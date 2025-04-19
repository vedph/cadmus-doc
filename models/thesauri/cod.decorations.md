---
title: "Thesauri - COD Decorations"
layout: default
parent: "Thesauri Examples"
nav_order: 22
---

# Thesauri - Codicology - Decorations

- 🌐 [part](https://github.com/vedph/cadmus-codicology/blob/master/docs/cod-decorations.md)
- 📚 `cod-decoration-flags`
- 📚 `cod-decoration-artist-types`
- 📚 `cod-decoration-artist-style-names`
- 📚 `cod-decoration-element-types`
- 📚 `cod-decoration-element-flags` (filtered)
- 📚 `cod-decoration-element-typologies` (filtered)
- 📚 `cod-decoration-element-colors` (filtered)
- 📚 `cod-decoration-element-gildings` (filtered)
- 📚 `cod-decoration-element-techniques` (filtered)
- 📚 `cod-decoration-element-tools` (filtered)
- 📚 `cod-decoration-element-positions` (filtered)
- 📚 `cod-image-types`
- `assertion-tags`
- `chronotope-tags`
- `doc-reference-tags`
- `doc-reference-types`

The decorations part uses some thesauri to dynamically update its UI according to data being entered. The core thesaurus here is 📚 `cod-decoration-element-types`, required, which defines the types of decoration elements, such as initial pages, illustrations, headletters, paragraphematic signs, etc.

The UI structure is hierarchical:

- list of decorations:
  - edited decoration:
    - ID, name, features (📚 `cod-decoration-flags`), note, chronotopes, references, artists, list of elements:
      - edited element

In the edited elements, when the edited element type is selected from the list of element types:

- the thesauri entries marked as "filtered" are filtered according to the selected type. To this end, the IDs of these thesauri follow the syntax of hierarchical thesauri, e.g. `par.rubrication` is the entry for "rubrication" which must be displayed only when the element type is `par`.
- some areas of the UI are hidden according to the selected type. To this end, the 📚 `cod-decoration-type-hidden` thesaurus contains a list of identifiers from `cod-decoration-element-types`, each having as value a space-delimited list of tokens. In this thesaurus, each token represents a portion of the UI which should be hidden when that element type is selected. For instance, the paragraphematic type has ID `par` in the types thesaurus; in the hidden thesaurus, an entry has `par` as its ID, and `subject textRelation lineHeight` as its value. The value contains 3 UI portions to hide, each identified by a name.

The portions which can be hidden are:

- flags
- typologies
- subject
- colors
- gildings
- techniques
- tools
- positions
- lineHeight
- textRelation
- refSign

```json
[
  {
    "id": "cod-decoration-artist-types@en",
    "entries": [
      {
        "id": "illuminator",
        "value": "miniatore"
      },
      {
        "id": "workshop",
        "value": "bottega"
      },
      {
        "id": "scribe",
        "value": "copista"
      },
      {
        "id": "other",
        "value": "altro"
      }
    ]
  },
  {
    "id": "cod-decoration-element-colors@en",
    "entries": [
      {
        "id": "blue",
        "value": "blu"
      },
      {
        "id": "red",
        "value": "rosso"
      },
      {
        "id": "green",
        "value": "verde"
      },
      {
        "id": "yellow",
        "value": "giallo"
      },
      {
        "id": "white",
        "value": "bianco"
      },
      {
        "id": "purple",
        "value": "viola"
      },
      {
        "id": "turquoise",
        "value": "turchese"
      },
      {
        "id": "other",
        "value": "altro"
      }
    ]
  },
  {
    "id": "cod-decoration-element-flags@en",
    "entries": [
      {
        "id": "pag-inc.blank",
        "value": "spazi bianchi"
      },
      {
        "id": "pag-inc.sketch",
        "value": "incompiuto"
      },
      {
        "id": "pag-inc.ill",
        "value": "illustrazione"
      },
      {
        "id": "pag-inc.orn",
        "value": "ornamentazione"
      },
      {
        "id": "pag-inc.ini",
        "value": "iniziali"
      },
      {
        "id": "pag-dec.blank",
        "value": "spazi bianchi"
      },
      {
        "id": "pag-dec.sketch",
        "value": "incompiuto"
      },
      {
        "id": "pag-dec.ill",
        "value": "illustrazione"
      },
      {
        "id": "pag-dec.orn",
        "value": "ornamentazione"
      },
      {
        "id": "pag-dec.ini",
        "value": "iniziali"
      },
      {
        "id": "ill.blank",
        "value": "spazio bianco"
      },
      {
        "id": "ill.sketch",
        "value": "incompiuto"
      },
      {
        "id": "orn.blank",
        "value": "spazio bianco"
      },
      {
        "id": "orn.sketch",
        "value": "incompiuto"
      },
      {
        "id": "dia.blank",
        "value": "spazi bianchi"
      },
      {
        "id": "dia.sketch",
        "value": "incompiuto"
      },
      {
        "id": "ini-pla.blank",
        "value": "spazio bianco"
      },
      {
        "id": "ini-pla.sketch",
        "value": "incompiuto"
      },
      {
        "id": "ini-pla.guide",
        "value": "lettere guida"
      },
      {
        "id": "ini-wat.blank",
        "value": "spazio bianco"
      },
      {
        "id": "ini-wat.sketch",
        "value": "incompiuto"
      },
      {
        "id": "ini-wat.guide",
        "value": "lettere guida"
      },
      {
        "id": "ini-orn.blank",
        "value": "spazio bianco"
      },
      {
        "id": "ini-orn.sketch",
        "value": "incompiuto"
      },
      {
        "id": "ini-orn.guide",
        "value": "lettere guida"
      },
      {
        "id": "ini-fig.blank",
        "value": "spazio bianco"
      },
      {
        "id": "ini-fig.sketch",
        "value": "incompiuto"
      },
      {
        "id": "ini-fig.guide",
        "value": "lettere guida"
      },
      {
        "id": "ini-his.blank",
        "value": "spazio bianco"
      },
      {
        "id": "ini-his.sketch",
        "value": "incompiuto"
      },
      {
        "id": "ini-his.guide",
        "value": "lettere guida"
      },
      {
        "id": "par.blank",
        "value": "spazio bianco"
      },
      {
        "id": "par.sketch",
        "value": "incompiuto"
      }
    ]
  },
  {
    "id": "cod-decoration-element-gildings@en",
    "entries": [
      {
        "id": "leaf",
        "value": "oro in foglia"
      },
      {
        "id": "powder",
        "value": "oro a conchiglia"
      },
      {
        "id": "other",
        "value": "altre lamine metall."
      },
      {
        "id": "punch-marks",
        "value": "punzonature"
      },
      {
        "id": "sgraffito",
        "value": "incisioni"
      },
      {
        "id": "stippled",
        "value": "graniture"
      }
    ]
  },
  {
    "id": "cod-decoration-element-positions@en",
    "entries": [
      {
        "id": "ill.full-page",
        "value": "a piena pagina"
      },
      {
        "id": "ill.interpol",
        "value": "intercalata"
      },
      {
        "id": "ill.intercol",
        "value": "intercolumnio"
      },
      {
        "id": "ill.margin-e",
        "value": "margine est."
      },
      {
        "id": "ill.margin-i",
        "value": "margine int."
      },
      {
        "id": "ill.margin-t",
        "value": "margine sup."
      },
      {
        "id": "ill.foot",
        "value": "margine inf."
      },
      {
        "id": "ill.other",
        "value": "altro"
      },
      {
        "id": "dia.-",
        "value": ""
      },
      {
        "id": "ini-pla.aligned",
        "value": "allineata"
      },
      {
        "id": "ini-pla.inside",
        "value": "annegata"
      },
      {
        "id": "ini-pla.hanging",
        "value": "sporgente"
      },
      {
        "id": "ini-wat.aligned",
        "value": "allineata"
      },
      {
        "id": "ini-wat.hanging",
        "value": "sporgente"
      },
      {
        "id": "ini-wat.inside",
        "value": "annegata"
      },
      {
        "id": "ini-orn.aligned",
        "value": "allineata"
      },
      {
        "id": "ini-orn.hanging",
        "value": "sporgente"
      },
      {
        "id": "ini-orn.inside",
        "value": "annegata"
      },
      {
        "id": "ini-fig.aligned",
        "value": "allineata"
      },
      {
        "id": "ini-fig.hanging",
        "value": "sporgente"
      },
      {
        "id": "ini-fig.inside",
        "value": "annegata"
      },
      {
        "id": "ini-his.aligned",
        "value": "allineata"
      },
      {
        "id": "ini-his.hanging",
        "value": "sporgente"
      },
      {
        "id": "ini-his.inside",
        "value": "annegata"
      },
      {
        "id": "orn.-",
        "value": ""
      },
      {
        "id": "par.-",
        "value": ""
      }
    ]
  },
  {
    "id": "cod-decoration-element-techniques@en",
    "entries": [
      {
        "id": "watercolor",
        "value": "acquerello"
      },
      {
        "id": "gouache",
        "value": "tempera"
      },
      {
        "id": "ink",
        "value": "inchiostro"
      },
      {
        "id": "other",
        "value": "altro"
      }
    ]
  },
  {
    "id": "cod-decoration-element-tools@en",
    "entries": [
      {
        "id": "pen",
        "value": "penna"
      },
      {
        "id": "brush",
        "value": "pennello"
      },
      {
        "id": "other",
        "value": "altro"
      }
    ]
  },
  {
    "id": "cod-decoration-element-types@en",
    "entries": [
      {
        "id": "pag-inc",
        "value": "pagina incipitaria"
      },
      {
        "id": "pag-dec",
        "value": "pagina decorata"
      },
      {
        "id": "ill",
        "value": "illustrazione"
      },
      {
        "id": "orn",
        "value": "ornamentazione"
      },
      {
        "id": "dia",
        "value": "diagramma"
      },
      {
        "id": "ini-pla",
        "value": "iniziali - semplici"
      },
      {
        "id": "ini-wat",
        "value": "iniziali - filigranate"
      },
      {
        "id": "ini-orn",
        "value": "iniziali - ornate"
      },
      {
        "id": "ini-fig",
        "value": "iniziali figurate"
      },
      {
        "id": "ini-his",
        "value": "iniziali istoriate"
      },
      {
        "id": "par",
        "value": "peritestuali"
      },
      {
        "id": "wsp",
        "value": "spazi bianchi"
      }
    ]
  },
  {
    "id": "cod-decoration-element-typologies@en",
    "entries": [
      {
        "id": "pag-inc.architectural",
        "value": "frontespizio architettonico"
      },
      {
        "id": "pag-dec.frontispiece",
        "value": "antiporta"
      },
      {
        "id": "pag-dec.architectural",
        "value": "pag-dec.frontespizio architettonico"
      },
      {
        "id": "ill.miniature",
        "value": "miniatura"
      },
      {
        "id": "ill.drawing",
        "value": "disegno"
      },
      {
        "id": "ill.tabular",
        "value": "tabellare"
      },
      {
        "id": "ill.papyrus",
        "value": "campo libero"
      },
      {
        "id": "ini-orn.braided",
        "value": "a cappio intrecciato"
      },
      {
        "id": "ini-orn.zoomrp",
        "value": "zoomorfa"
      },
      {
        "id": "ini-orn.phytomrp",
        "value": "fitomorfa"
      },
      {
        "id": "ini-orn.anthropomrp",
        "value": "antropomorfa"
      },
      {
        "id": "ini-orn.prism",
        "value": "prismatica"
      },
      {
        "id": "ini-orn.tendrils",
        "value": "bianchi girari"
      },
      {
        "id": "ini-orn.kaleidoscopic",
        "value": "caleidoscopica"
      },
      {
        "id": "ini-orn.geometric",
        "value": "geometrica"
      },
      {
        "id": "ini-orn.other",
        "value": "altro"
      },
      {
        "id": "ini-wat.split1",
        "value": "fessa"
      },
      {
        "id": "ini-wat.split2",
        "value": "rifessa"
      },
      {
        "id": "orn.clipeus",
        "value": "clipeo"
      },
      {
        "id": "orn.frame-arch",
        "value": "cornice architettonica"
      },
      {
        "id": "orn.frieze-swirl-w",
        "value": "fregio a bianchi girari"
      },
      {
        "id": "orn.frieze-swirl-w-fig",
        "value": "fregio a bianchi girari figurato"
      },
      {
        "id": "orn.frieze-swirl-w-hist",
        "value": "fregio a bianchi girari istoriato"
      },
      {
        "id": "orn.frieze-fig",
        "value": "fregio figurato"
      },
      {
        "id": "orn.frieze-hist",
        "value": "fregio istoriato"
      },
      {
        "id": "orn.frieze-orn",
        "value": "fregio ornamentale"
      },
      {
        "id": "orn.coat",
        "value": "stemma"
      },
      {
        "id": "orn.tab",
        "value": "tabula ansata"
      },
      {
        "id": "orn.other",
        "value": "altro"
      },
      {
        "id": "par.run-heads",
        "value": "titoli correnti"
      },
      {
        "id": "par.rubrication",
        "value": "rubricatura"
      },
      {
        "id": "par.paragraph",
        "value": "segni di paragrafo"
      },
      {
        "id": "par.cartulation",
        "value": "cartulazione"
      },
      {
        "id": "par.call-dec",
        "value": "richiami decorati"
      },
      {
        "id": "par.call-fig",
        "value": "richiami figurati"
      },
      {
        "id": "par.other",
        "value": "altro"
      }
    ]
  },
  {
    "id": "cod-decoration-flags@en",
    "entries": [
      {
        "id": "not-original",
        "value": "non originale"
      },
      {
        "id": "incomplete",
        "value": "incompleta"
      }
    ]
  },
  {
    "id": "cod-decoration-type-hidden@en",
    "entries": [
      {
        "id": "pag-inc",
        "value": "typologies positions lineHeight refSign"
      },
      {
        "id": "pag-dec",
        "value": "typologies positions lineHeight refSign"
      },
      {
        "id": "ill",
        "value": "lineHeight refSign"
      },
      {
        "id": "orn",
        "value": "lineHeight refSign"
      },
      {
        "id": "dia",
        "value": "lineHeight refSign"
      },
      {
        "id": "ini-pla",
        "value": "typologies subject textRelation"
      },
      {
        "id": "ini-wat",
        "value": "subject textRelation"
      },
      {
        "id": "ini-orn",
        "value": "textRelation"
      },
      {
        "id": "par",
        "value": "textRelation lineHeight"
      },
      {
        "id": "wsp",
        "value": "colors gildings typologies techniques subject textRelation tools lineHeight refSign"
      }
    ]
  }
]
```
