# Caută în timp ce scrii

O posibilă variantă ar fi abuzarea lui `slop`.

```json
{
  "query": {
    "match_phrase_prefix": {
      "title": {
        "query": "războiul stelelor",
        "slop": 10
      }
    }
  }
}
```
