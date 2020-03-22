# Function score query

Această interogare, `function_score` permite modificarea scorului documentelor găsite.

```json
{
    "query": {
        "function_score": {
            "query": { "match_all": {} },
            "boost": "5",
            "random_score": {},
            "boost_mode":"multiply"
        }
    }
}
```

Mai mult, se pot combina mai multe funcții. În astfel de cazuri, se poate opta pentru aplicarea funcției doar dacă documentul satisface un anumit filtru.

```json
{
    "query": {
        "function_score": {
          "query": { "match_all": {} },
          "boost": "5",
          "functions": [
              {
                  "filter": { "match": { "test": "bar" } },
                  "random_score": {},
                  "weight": 23
              },
              {
                  "filter": { "match": { "test": "cat" } },
                  "weight": 42
              }
          ],
          "max_boost": 42,
          "score_mode": "max",
          "boost_mode": "multiply",
          "min_score" : 42
        }
    }
}
```

Folosind `"boost": "5"` crești scorul documentului.

În ceea ce privește `score_mode` poate avea niște valori care specifică cum sunt combinate scorurile calculate.

- `multiply` - scorul este multiplicat (default);
- `sum` - scorurile sunt adunate;
- `avg` - scorurile sunt reduse la medie;
- `first` - prima funcție care are un filtru care s-a potrivit;
- `max` - este folosit scorul maxim;
- `min` - este folosit scorul minim.
