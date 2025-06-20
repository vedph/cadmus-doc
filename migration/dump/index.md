---
title: "Dumping"
layout: default
parent: "Migration"
nav_order: 1
---

# Dumping

Dumping Cadmus data is a process designed for migrating data either fully or incrementally, but also to provide logic for time frame-based data browsing.

The process relies on two main components: a data framer and a data dumper.

The task of the _data framer_ is to "frame" a subset of items, filtered by any item's (and possibly part's) metadata, including their modified time. This type of filter provides a time frame where you can look at any item with its parts in the state they were within the specified period. This is useful not only to export data, especially in an incremental way; but also to provide the logic for viewing the evolution of each item in time.

The _data dumper_ relies on the data framer to select the items (and their parts) to export. Currently the only dumper draws data from the MongoDB database and produces JSON dumps.

## Leveraging History

The source Cadmus database contains among others these essential collections:

- **items**: a list of fixed-schema records with essential metadata about each item. Items have among other properties `_id`
  (a GUID), `timeCreated`, `timeModified`.
- **parts**: a list of varialble-schema records with some fixed essential metadata about each part. Parts have among other properties `_id` (a GUID), `timeCreated`, `timeModified`, and an `itemId`, which works like a foreign key to link that part to a specific item.
- **history-items**: items editing history.
- **history-parts**: parts editing history.

History collections are used to store copies of items and parts, whenever they get saved in the database during editing. When this happens, a copy of the item/part is stored in the corresponding history collection: the entry has its own `_id` (a GUID), and the GUID of its source item/part in `referenceId`.

Also, there is a `status` numeric field with values 0=created, 1=updated, 2=deleted. When an item/part is first created, a copy of it is stored in history with status=created; then, on each successive update, a copy of it is stored in history with
status=updated. If it gets deleted, the item/part is removed from its collection (`items` or `parts`), but a copy of it before deletion is stored in the corresponding history part, with status=deleted.

Thus, in a sense the full dataset is in the history collections, while `items` and `parts` just include the currently active items and parts, without the deleted ones, and updated to their latest versions. That's why the data framer uses history collections as its data source, and typically applies a time filter to frame a specific time window to look at data.

The framer works as follows:

1. filter items as requested. This filters by any criteria, including minimum and maximum modified time, which provide the time frame when there is one. Otherwise the time frame just corresponds to the whole dataset.
2. group items by `referenceId`. This means grouping history entries by item. Each group is the full history of the item within the boundaries defined by filter.
3. for each group, select the latest entry. This is the item we want, as it corresponds to the item which was active within the given time frame.
4. for each selected item, collect item parts and add them to a new `_parts` array property representing the item. Also set its `_id` equal to `_referenceId` and remove `_referenceId`, renaming `status` into `_status`.

Getting item parts works in the same way: filter history parts (including the item ID filter for the item being
processed), group by `referenceId`, select the latest entry from each group, and return the part with an adjusted schema.

## Full vs Incremental

The data framer can be configured to work for either full or incremental selection. In incremental mode, it only includes items or parts changed (created, updated, or deleted) in the specified time frame. For an item or part to be included, we require that it was changed in that specific time frame, not just before, nor of course after. So, if an item was never changed after its creation, or was changed but before the time frame, and it is still active (i.e. it was not deleted), it will _not_ be selected. If instead it was updated or deleted within the time frame, it will be selected.

This behavior is useful to provide incremental dumps, which allow a third-party system progressively import new data without having to re-import the whole database each time.

In full mode instead, the framer includes all active (or deleted) items or parts, as of the end of the time frame. So, all the active items, even when they were never changed, are selected.

For instance, consider this set: let's represent items with `i` followed by a number, and their parts with `p` followedby a number. The item each part belongs to is specified in brackets. Columns refer to a date, which for short just contains month and day of the same year. In the cells, `C`=created, `U`=updated, `D`=deleted. So, the table provides an overview of the evolution of data over time:

| obj     | 01-01 | 01-15 | 02-01 | 02-02 | 03-01 | 03-15 | 04-01 | 05-01 |
| ------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| i1      | CU    |       |       |       |       |       |       |       |
| i2      |       |       | CU    |       | U     |       |       |       |
| i3      |       |       |       |       |       |       | CU    |       |
| i4      |       | CU    |       |       |       |       |       | D     |
| p1 (i1) | CU    |       |       |       |       |       |       |       |
| p2 (i2) |       |       | CU    |       |       |       |       |       |
| p3 (i2) |       |       |       | CU    |       | U     |       |       |
| p4 (i3) |       |       |       |       |       |       | CU    |       |
| p5 (i4) |       | CU    |       |       |       |       |       | D     |

> Note that `C` represents `timeCreated` and `M` `timeModified`. All these properties are always present for each item or part. At the time of creation, they are equal. When the object is changed, `timeModified` is updated, while `timeCreated` stays unchanged forever. The framer relies on `M`, but here we add `C` to clearly show that the item was created at a given point in time. That's why you have `CU` which means that the item was created at that time.

In _full mode_, if we set no filters all the items with their parts will be exported, including those which were deleted (marked as such in their `_status`), unless we ask the framer to exclude them.

If instead we set a max modified date equal to 03-01, the framer will include:

- i1 (created 01-01), with part p1;
- i2 (updated 03-01), with parts p2 and p3 (with the values current before its update on 03-15);
- i4 (deleted 05-01), with part p5.

Yet, it will exclude:

- i3 (created 04-01), because it was created after the considered time frame.

Let us now consider this other dataset (we use bold for selected objects):

| obj         | 01-01 | 01-15 | 02-01 | 02-02 | 03-01 | 03-15 | 04-01 | 05-01 | 05-10 | 05-15 |
| ----------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| i1          | CU    |       |       |       |       |       |       |       |       |       |
| **i2**      |       |       | CU    |       | U     |       |       |       |       |       |
| i3          |       |       |       |       |       |       | CU    |       |       |       |
| **i4**      |       | CU    |       |       |       |       |       | D     |       |       |
| **i5**      |       |       |       |       |       |       |       |       |       | CU    |
| p1 (i1)     | CU    |       |       |       |       |       |       |       |       |       |
| **p2** (i2) |       |       | CU    |       |       |       |       |       |       |       |
| **p3** (i2) |       |       |       | CU    |       | U     |       |       | U     |       |
| p4 (i3)     |       |       |       |       |       |       | CU    |       |       |       |
| **p5** (i4) |       | CU    |       |       |       |       |       | D     |       |       |
| **p6** (i5) |       |       |       |       |       |       |       |       |       | CU    |

In _incremental mode_, say we specify a time frame from 04-15 to 05-20. This will select:

- i2, because even if it was not directly changed in the specified time frame, its part p3 was updated on 05-10. A change in any of the item's parts counts as a change in the item itself, except when requested otherwise. Also note that this p3 will reflect the updates happened on 05-10, rather than be equal to its state on 02-02.
- i4, because it was deleted on 05-01, which is within the time frame.
- i5, because it was created on 05-15, which is within the time frame.
