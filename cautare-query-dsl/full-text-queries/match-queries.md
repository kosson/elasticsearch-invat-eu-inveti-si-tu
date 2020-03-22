# Match queries

Aceste interogări returnează documente în care se află un anumit text, număr sau dată. Textul introdus este mai întâi analizat.
Acest tip de interogare este unui standard pentru căutările full-text, incluzând opțiuni pentru căutarea fuzzy.

```json
{
    "query": {
        "match" : {
            "message" : {
                "query" : "this is a test"
            }
        }
    }
}
```

Parametrul top-level este numele câmpului în care se va face căutarea.

## Parametrii lui `<field>`

### `query`

Poate fi un fragment de text, un număr, un boolean care sunt necesare și pe care dorești să le cauți în câmpul menționat. Înainte de a face o căutare, textul după care se va face căutarea va fi analizat. Acest lucru înseamnă că o căutare se va face după tokenii reieșiți din analiză, nu după termenii exacți.

### `analyzer`

Este numele analizorului folosit pentru a tranforma textul după care se va face căutarea în tokeni. Valoarea din oficiu pentru acest analizor este cel folosit la momentul în care s-a făcut indexarea.

### `auto_generate_synonyms_phrase_query`

Este un boolean opțional. Dacă are valoarea `true`, sunt create automat interogări `match phrases` pentru sinonime care au mai mulți termeni. Valoarea din oficiu este `true`.

### `fuzziness`

Este un string care poate fi introdus opțional. Aceasta este distanța maximă cu care numărul de caractere poate diferi față de căutarea exactă a termenului.

## Resurse

- https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness
- https://en.wikipedia.org/wiki/Levenshtein_distance
