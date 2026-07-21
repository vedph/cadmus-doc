# Thesauri - Historical Events

- 🌐 [part](https://github.com/vedph/cadmus-general/blob/master/docs/historical-events.md) (role-dependent)
- 📚 `event-relations`
- 📚 `event-tags`
- 📚 `event-types`
- 📚 `pin-link-settings`
- `assertion-tags`
- `chronotope-tags`
- `doc-reference-tags`
- `doc-reference-types`
- `pin-link-scopes`
- `pin-link-tags`

```json
[
  {
    "id": "event-types_bio@en",
    "entries": [
      {
        "id": "person.birth",
        "value": "person: birth"
      },
      {
        "id": "person.death",
        "value": "person: death"
      },
      {
        "id": "person.marriage",
        "value": "person: marriage"
      }
    ]
  },
  {
    "id": "event-relations_bio@en",
    "entries": [
      {
        "id": "person:birth:mother",
        "value": "persona: nascita: madre"
      },
      {
        "id": "person:birth:father",
        "value": "persona: nascita: padre"
      }
    ]
  },
  {
    "id": "pin-link-settings@en",
    "entries": [
      {
        "id": "switch-mode",
        "value": "true"
      }
    ]
  },
  {
    "id": "pin-link-settings_bio@en",
    "targetId": "pin-link-settings"
  }
]
```
