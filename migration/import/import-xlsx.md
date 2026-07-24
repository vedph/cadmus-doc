# Importing from Excel

This procedure is an example of importing items from a spreadsheet into a Cadmus database via the Proteus conversion framework.

## Leveraging Proteus

In short, Proteus adapts any input source to a list of so-called decoded entries. A **decoded entry** is the smallest meaningful unit of input for conversion. There are three types:

- `T` (text entry) — raw text fragments
- `P` (property entry) — simple style properties (bold, italic, color, etc.)
- `C` (command entry) — structural or metatextual commands with arguments

Example of a decoded paragraph with world in bold:

```txt
C block(open=1)
T Hello, 
P bold=1
T world
P bold=0
T !
C block(open=0)
```

In this case, assuming that each record to import is a row in the source spreadsheet, each row is decoded into a set of entries:

1. for each sheet: `C sheet-start(index=INDEX, n=NAME)` ... `C sheet-end()`.
2. for each row:  `row-start(y=N)` ... `C row-end()`.
3. for each column in each row: `C col-start(x=N, n=COLUMN_NAME)`, `T ...`, `C col-end()` where the text entry contains the cell's value.

For instance:

```txt
C sheet-start(index=0, n=Foglio1)

C row-start(y=2)

C col-start(x=1, n=object_name)
T ih00232000_00200777_a2r_(1)
C col-end()

C col-start(x=2, n=folio)
T a2r
C col-end()

C col-start(x=3, n=object_measures_(h_x_w))
T 48x75
C col-end()
```

This list of entries represents the first 3 cells of the first row of the first sheet in a source spreadsheet. So, the idea is to parse one row at a time, and within it one cell at a time; for each cell type, a specific logic can be used.

A stock Proteus region detector is then used to define a region for each sheet, row, and column. Column region names start with col- followed by the column's label (where spaces are converted to `_` and text is lowercased).

This allows for a highly modular approach, where you just create a region parser for each region (=column) you want to import. Typically, each parser builds a part and adds it to the item corresponding to the current row; or just adds more metadata to the item; or build some temporary structure to be progressively completed while combining multiple cells together.

In the end, the item built can be exported both for dumping and for import purposes, using e.g. these exporters:

- an Excel dump exporter, to see all the decoded entries with their regions.
- a Markdown dump exporter, to see all the items with their parts.
- a MongoDB exporter, to store the items in a target Cadmus database.
