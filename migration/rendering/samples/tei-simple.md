---
title: "Sample: Simple TEI" 
layout: default
parent: "Rendering Configuration"
nav_order: 1
---

# Rendering Sample - Simple TEI

This example shows a TEI rendition of the simplest critical text scenario: just a text and its critical apparatus, with no additional layers.

The configuration follows:

```json
{
  "ItemIdCollector": {
    "Id": "it.vedph.item-id-collector.mongo",
    "Options": {
      "FacetId": "text"
    }
  },
  "ContextSuppliers": [
    {
      "Keys": "flags",
      "Id": "it.vedph.renderer-context-supplier.flag",
      "Options": {
        "On": {
          "8": "block-type=poetry"
        },
        "Off": {
          "8": "block-type=prose"
        }
      }
    }
  ],
  "TextTreeFilters": [
    {
      "Keys": "block-linear",
      "Id": "it.vedph.text-tree-filter.block-linear"
    }
  ],
  "RendererFilters": [
    {
      "Keys": "nl-appender",
      "Id": "it.vedph.renderer-filter.appender",
      "Options": {
        "Text": "\r\n"
      }
    },
    {
      "Keys": "ns-remover",
      "Id": "it.vedph.renderer-filter.replace",
      "Options": {
        "Replacements": [
          {
            "Source": " xmlns=\"http://www.tei-c.org/ns/1.0\"",
            "Target": "",
            "Repetitions": 1
          }
        ]
      }
    }
  ],
  "TextPartFlatteners": [
    {
      "Keys": "it.vedph.token-text",
      "Id": "it.vedph.text-flattener.token"
    }
  ],
  "TextTreeRenderers": [
    {
      "Keys": "tei",
      "Id": "it.vedph.text-tree-renderer.tei-app-linear",
      "Options": {
        "FilterKeys": ["nl-appender", "ns-remover"],
        "ZeroVariantType": "omissio",
        "BlockElements": {
          "default": "tei:p",
          "poetry": "tei:l",
          "prose": "tei:p"
        }
      }
    }
  ],
  "ItemComposers": [
    {
      "Keys": "default",
      "Id": "it.vedph.item-composer.tei.fs",
      "Options": {
        "ContextSupplierKeys": ["flags"],
        "TextPartFlattenerKey": "it.vedph.token-text",
        "TextTreeFilterKeys": ["block-linear"],
        "TextTreeRendererKey": "tei",
        "TextHead": "<?xml version=\"1.0\" encoding=\"utf-8\"?>\n<TEI xmlns=\"http://www.tei-c.org/ns/1.0\">\n  <teiHeader>\n    <fileDesc>\n      <titleStmt>\n        <title>Sidonius</title>\n      </titleStmt>\n      <publicationStmt>\n        <p>Not published.</p>\n      </publicationStmt>\n      <sourceDesc>\n        <p>Undisclosed.</p>\n      </sourceDesc>\n    </fileDesc>\n  </teiHeader>\n  <text>\n    <body>\n      <div>\n",
        "TextTail": "      </div>\n    </body>\n  </text>\n</TEI>",
        "OutputDirectory": "c:\\users\\dfusi\\Desktop\\sidon"
      }
    }
  ]
}
```

From top to bottom:

1. the **item ID collector** draws data from a Mongo database (the default for Cadmus), and filters items to select only those representing text items (whose facet is `text`).
2. a flag-based **context supplier** is used to supply the context type for the text item: in the editor, a flag (whose value here happens to be 8) is used to mark those text items written in poetry. This supplies more metadata to the rendering context: when the flag is on, block type is set to poetry; when it is off, it is set to prose.
3. a **tree text filter** is used to split nodes containing newlines in their text, so we can be sure that newlines do not appear in text and each node before newline is marked as such.
4. two renderer filters are applied:
   - `nl-appender` is used to append a CR-LF pair after each item rendition. This provides a tidier TEI code.
   - `ns-remover` is used to remove the redundant namespace in the TEI output by the tree renderer. There, the namespace is required because the rendered output is a standalone TEI fragment which requires a namespace at its first-level element(s); but once in the context of a full document, this namespace would be redundant.
5. a **text part flattener** is used to flatten the annotated text of token-based text parts.
6. a **text tree renderer** is used to render a token-based text with a single apparatus layer into a simple TEI with embedded apparatus.
7. the **item composer** orchestrates all these components storing results into files. It uses:
   - the above defined context supplier (nr.2) for detecting poetry.
   - the above defined flattener (nr.5) to flatten layers of annotations on a single linear sequence of text segments. In this example we just have a single layer, so the segmentation will just result from combining the base text with the apparatus annotations on its top.
   - the above defined tree filter (nr.3) to properly handle newlines in text.
   - the above defined tree renderer (nr.6) to render text and apparatus into TEI.
   - a preset text head with all the markup before the text body.
   - a preset text tail with all the markup after the text body.
   - an output directory.

For a single item, this produces an output like this (I reformatted it to make it more readable here):

```xml
<?xml version="1.0" encoding="utf-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
    <teiHeader>
        <fileDesc>
            <titleStmt>
                <title>Sidonius</title>
            </titleStmt>
            <publicationStmt>
                <p>Not published.</p>
            </publicationStmt>
            <sourceDesc>
                <p>Undisclosed.</p>
            </sourceDesc>
        </fileDesc>
    </teiHeader>
    <text>
        <body>
            <div>
                <p source="$db66b931-d468-4478-a6ae-d9e56e9431b9" n="1">1. SIDONIUS PETRONIO SUO
                    SALUTEM</p>
                <p source="$0ac3a272-f1bd-4e9f-b5f5-7c1f76f19eea" n="1">Tu quidem pulchre (mos hic
                    tuus, et persevera), vir omnium bonorum, qui uspiam degunt, laude dignissime,
                    quod amicorum gloriae, sicubi locus, lenocinaris. Hinc est quod etiam scrinia
                    Arverna petis eventilari, cui sufficere suspicabamur, si quid superiore vulgatu
                    protulissemus. Itaque morem geremus iniunctis, actionem tamen stili eatenus
                    prorogaturi, ut epistularum seriem nimirum a primordio voluminis inchoatarum in
                    extimo fine parvi adhuc numeri summa protendat, opus videlicet explicitum quodam
                    quasi marginis sui limbo coronatura.</p>
                <p source="$7080b0e1-f5ba-434b-a28d-a176afc10c0c" n="1">Sed plus cavendum est, ne
                    sera propter iam propalati augmenta voluminis in aliquos forsitan incidamus
                    vituperones, quorum fugere linguas cote livoris naturalitus acuminatas ne
                    Demosthenis quidem Ciceronisque sententiae artifices et eloquia fabra potuere,
                    quorum anterior <app n="1"><rdg n="1"
                            wit="#f1 #f2 #f3 #l #m #p2 #p3 #pr #r #s #v1 #v2 #v3 #v4"
                            >orator</rdg><lem n="2" wit="#a #b #h #lon #p4">orator</lem><rdg n="3"
                            wit="#o">oratus</rdg><rdg n="4" wit="#p1">oratur</rdg></app> Demaden,
                    citerior Antonium toleravere derogatores; qui lividi cum fuerint malitiae
                    clarae, dictionis obscurae, tamen ad notitiam posterorum per odia virtutum <app
                        n="2"><rdg n="1" xml:id="rdg1" wit="#f1 #o #p1 #r #v1"
                            >decucurrerunt</rdg><witDetail target="#rdg1" wit="#f1"
                            >a.c.</witDetail><witDetail target="#rdg1" wit="#p1"
                            >a.c.</witDetail><rdg n="2" xml:id="rdg2"
                            wit="#a #b #f1 #f2 #f3 #h #l #lon #m #p1 #p2 #p3 #p4 #pr #s #v2 #v3 #v4"
                            >decurrerunt</rdg><witDetail target="#rdg2" wit="#f1"
                            >p.c.</witDetail><witDetail target="#rdg2" wit="#p1"
                        >p.c.</witDetail></app>.</p>
                <p source="$1bf00c75-f2c8-4d17-ada9-cbfa4fc9e83b" n="1">Sed quia hortaris, repetitis
                        <app n="1"><rdg n="1"
                            wit="#a #b #f1 #f3 #h #l #lon #m #o #p2 #p3 #p4 #pr #r #s #v1 #v2 #v3 #v4"
                            >laxemus</rdg><rdg n="2" wit="#f2 #p1">laxamus</rdg></app> vela <app
                        n="2"><rdg n="1" xml:id="rdg3"
                            wit="#a #b #f1 #f2 #f3 #h #l #lon #m #p1 #p2 #p3 #p4 #pr #r #s #v1 #v2 #v3 #v4"
                            >turbinibus</rdg><witDetail target="#rdg3" wit="#p1"
                            >p.c.</witDetail><rdg n="2" xml:id="rdg4" wit="#o #p1"
                            >turbidinis</rdg><witDetail target="#rdg4" wit="#p1"
                        >a.c.</witDetail></app> et qui veluti maria transmisimus, hoc quasi stagnum
                    pernavigemus. Nam satis habeo deliberatum, sicut <app n="3"><rdg n="1"
                            xml:id="rdg5"
                            wit="#a #b #f1 #f2 #f3 #l #lon #m #o #p1 #p2 #pr #r #s #v1 #v2 #v3 #v4"
                            >adhibendam</rdg><witDetail target="#rdg5" wit="#lon"
                            >p.c.</witDetail><rdg n="2" xml:id="rdg6" wit="#h #lon #p4"
                            >adhibendum</rdg><witDetail target="#rdg6" wit="#lon"
                            >a.c.</witDetail><rdg n="3" wit="#p3">adibendum</rdg></app> in
                    conscriptione diligentiam, ita tenendam in editione constantiam. Demum vero
                    medium nihil est: namque aut minimum ex hisce metuendum est aut per omnia omnino
                        <app n="4"><rdg n="1" xml:id="rdg7"
                            wit="#a #b #f1 #f2 #f3 #h #l #lon #p1 #p2 #p3 #p4 #pr #r #s #v1 #v3 #v4"
                            >conticescendum</rdg><witDetail target="#rdg7" wit="#p1"
                            >p.c.</witDetail><rdg n="2" xml:id="rdg8" wit="#m #o #p1 #v2"
                            >conticiscendum</rdg><witDetail target="#rdg8" wit="#p1"
                            >a.c.</witDetail></app>. Vale.</p>
            </div>
        </body>
    </text>
</TEI>
```
