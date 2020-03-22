# Ajustarea căutărilor

## Proximity search

Aceste căutări permit existența altor termeni între cei pe care-i cauți.

```json
POST /proximity/_doc/1
{
  "titlu": "Ceva neașteptat"
}

POST /proximity/_doc/2
{
  "titlu": "Ceva total neașteptat"
}

POST /proximity/_doc/3
{
  "titlu": "Ceva total neașteptat și sincer"
}

POST /proximity/_doc/4
{
  "titlu": "Ceva total neașteptat (interesant)"
}

POST /proximity/_doc/5
{
  "titlu": "Ceva întunecat și neașteptat"
}

POST /proximity/_doc/6
{
  "titlu": "Ceva întunecat, negru, decrepit și neașteptat"
}

POST /proximity/_doc/7
{
  "titlu": "Neașteptat ceva"
}

# Prima interogare
GET /proximity/_search
{
  "query": {
    "match_phrase": {
      "titlu": "ceva neașteptat"
    }
  }
}
```

Adu-ți aminte de faptul că în indexul inversat sunt introduși termenii, dar este memorată și ordinea în care sunt termenii în document. Termenii din sintagma de căutare sunt trecuți prin același analizor folosit pentru constituirea indexului. Acesta este motivul pentru care la o căutare cu `match_phrase`, va fi găsit doar documentul cu indexul 1.

Pentru a indica faptul că termenii pot fi depărtați unii de ceilalți, se va introduce proprietatea `slop` cu o valoare care va indica distanța dintre termeni.

```json
GET /proximity/_search
{
  "query": {
    "match_phrase": {
      "titlu": {
        "query": "ceva neașteptat",
        "slop": 1
      }
    }
  }
}
```

Va aduce un rezultat cu patru din cele cinci. Cel de-al cincilea are doi termeni între: `Ceva întunecat și neașteptat`. Dacă valoarea lui slop ar fi 2, atunci ar fi permisă și permutarea termenilor. Pentru un slop de 2, vom găsit toate documentele care au termenii din `query` indiferent ce este între ei. Acesta este motivul pentru care în setul rezultatelor vom avea și documentul cu id-ul 7 în care termenii sunt inversați.
Câtă vreme nu este depășit numprul indicat de slop, termenii pot fi rotiți între ei.

Cu cât termenii sunt mai apropiați în documentele căutate, cu atât vor fi scorul mai bun. Totuși, nu uita că scorul este totuși influiențat de mulți alți factori. Pentru a putea influiența scorul privind relevanța, vom transforma `match_phrase` într-un `bool`. Adu-ți aminte că în cazul unui `bool`, din oficiu termenii sunt evaluați cu OR.

```json
GET /proximity/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "titlu": {
              "query": "ceva neașteptat"
            }
          }
        }
      ]
    }
  }
}
```

În cazul de mai sust, toate documentele vor fi în rezultat. Pentru a restrânge, vom folosi `should`. Acest should, dacă găsește document care să-l satisfacă, scorul va fi boosted pentru acelea.

```json
GET /proximity/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "titlu": {
              "query": "ceva neașteptat"
            }
          }
        }
      ],
      "should": {
        "match_phrase": {
          "titlu": {
            "query": "ceva neașteptat"
          }
        }
      }
    }
  }
}
```

## Fuzzy matches

Acest tip de căutări este folosit pentru a returna rezultate chiar dacă utilizatorul a scris greșit termenii de căutare.

```json
GET /movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Store Wars",
        "fuzziness": "auto"
      }
    }
  }
}
```

Reguli pentru auto:

- Dacă dimensiunea termenului este 1 sau 2, trebuie să se facă o regăsire exactă. fuzziness-ul nu este folosit;
- Dacă dimensiunea termenului este de 3 până la 5, valoarea este de 1;
- Ce-i mai mult de 5, va seta fuzziness-ul la 2.

O dimensiune mai mare ar taxa resursele, în primul rând și apoi, erorile umane sunt de unul sau două caractere. Ce depășește, dacă s-ar pune fuzziness-ul mai mare de 2, am avea o sumedenie de cuvinte găsite, ceea ce ar diminua intenția.

Dacă pui și un operator între termeni, iar fuzziness-ul la 1, poți avea chiar cei doi termeni greșit scris și tot va găsi documentul

```json
GET /movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Stor Ward",
        "operator": "and",
        "fuzziness": 1
      }
    }
  }
}
```

De cele mai multe ori setarea lui `fuzziness` la valoarea `auto` conduce la rezolvarea cea mai elegantă.

## Fuzzy queries

```json
GET /movies/_search
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "star",
        "fuzziness": "auto"
      }
    }
  }
}
```
