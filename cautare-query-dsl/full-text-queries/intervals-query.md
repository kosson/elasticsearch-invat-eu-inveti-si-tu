# Intervals query

Returnează documente în baza ordinii și apropierii termenilor căutați.
O interogare pe un interval (*intervals*) folosește câteva reguli de căutare în baza unui set de reguli. Aceste reguli se vor aplica termenilor dintr-un *field* specificat.

```json
{
  "query": {
    "intervals" : {
      "my_text" : {
        "all_of" : {
          "ordered" : true,
          "intervals" : [
            {
              "match" : {
                "query" : "my favorite food",
                "max_gaps" : 0,
                "ordered" : true
              }
            },
            {
              "any_of" : {
                "intervals" : [
                  { "match" : { "query" : "hot water" } },
                  { "match" : { "query" : "cold porridge" } }
                ]
              }
            }
          ]
        }
      }
    }
  }
}
```

În exemplul dat, sunt returnate documente care conțin fragmentul de text `my favourite food`, urmat imediat de fragmentul `hot water` sau `cold porridge` din câmpul `my_text`.
Căutarea va returna documentele a căror câmp `my_text` are valoarea `my favorite food is cold porridge` dar nu și cele care au valoarea `when it's cold my favorite food is porridge`.

Parametrii de top pentru o interogare `intervals` pe lângă `all_of` mai pot fi următoarele:

- `match`;
- `prefix`;
- `wildcard`;
- `any_of`.

## Regula cu `match`

Pentru acestă regulă poți avea următorii parametrii:

### `query`

Acesta este un string de text care trebuie menționat. Acesta este textul pe care ai dori să-l găsești în câmpul `my_text`.

```json
{
  "query": {
    "intervals": {
      "<nume_câmp>": {
        "match": {
          "query": "să existe acest fragment"
        }
      }
    }
  }
}
```

### `max_gaps`

Este un număr întreg opțional și reprezintă numărul de poziții între termenii după care se face căutarea. Termenii care apar totuși, dar mai departe de numărul specificat, nu sunt considerați ca fiind satisfacerea acestei reguli. Valoarea din oficiu este `-1`. Dacă este setat cu valoarea `0`, termenii trebuie să apară unul după altul.

### `ordered`

Este un boolean opțional. Dacă este setat cu valoarea `true`, termenii care sunt căutați trebuie să apară în ordinea specificată. Valoarea din oficiu este `0`.

### `analyzer`

Este un string care este numele unui analizator folosit pentru analiza termenilor din interogare. Valoarea sa din oficiu este cea a analizorului care a fost folosit pentru indexarea câmpului.

### `filter`

Este un obiect care definește un interval filter opțional.

### `use_field`

Acesta este un string opțional, care în momentul în care este menționat, se vor face căutări după acest match interval, fiind ignorat numele câmpului care a fost specificat mai sus în ierarhie. Acesta permite căutarea în mai multe câmpuri ca și cum toate ar fi același câmp.

## Regula cu `prefix`

Această regulă caută în termeni care încep cu un anumit set de caractere. Acest prefix poate îngloba până la 128 de termeni. Pentru a evita această limitare, se poate folosi opțiunea `index-prefixes`.

### `prefix`

Este un string necesar, fiind chiar caracterele termenilor pe care dorești să-i găsești în câmpul specificat anterior în ierarhie.

### `analyzer`

Este numele unui analizor folosit pentru a normaliza prefixul. Valoarea sa din oficiu este cea a analizorului care a fost folosit pentru indexarea câmpului.

### `use_field`

Este un string opțional, care dacă este specificat, atunci vor fi luate în considerare intervalele din acest câmp, nu cel defini mai sus în ierarhie.

## Regula `wildcard`

Acesta este o regulă care indică un pattern. Acest pattern poate fi extins să potrivească 128 de termeni posibili.

### `pattern`

Este un string care trebuie introdus și care definește un pattern (*șablon*) care să potrivească anumiți termeni.

Acest parametru oferă suport pentru doi operatori wildcard:
- `?` care potrivește orice caracter;
- `*` care potrizește zero sau mai multe caractere, incluzând pe cele blank.

Trebuie evitată introducerea operatorilor wildcard la începutul pattern-ului. Afectează performanța.

### `analyzer`

Este opțional, fiind numele unui analizor menționat pentru a normaliza pattern-ul. Valoarea sa din oficiu este cea a analizorului care a fost folosit pentru indexarea câmpului.

### `use_field`

Este un string opțional, care atunci când este menționat, căutarea se va face după acest interval și nu în câmpul menționat mai sus în ierarhie.

## Regula `all_of`

Face o căutare care poate fi o combinație de alte reguli.

### `intervals`

Este necesar să introduci un array de reguli care se combină. Un document trebuie să satisfacă toate regulile pentru a fi considerat.

### `max_gaps`

Este un număr întreg opțional și reprezintă numărul de poziții între termenii după care se face căutarea. Termenii care apar totuși, dar mai departe de numărul specificat, nu sunt considerați ca fiind satisfacerea acestei reguli. Valoarea din oficiu este `-1`. Dacă este setat cu valoarea `0`, termenii trebuie să apară unul după altul.

### `ordered`

Este un boolean opțional. Dacă este `true`, intervalele produse de aceaste reguli ar trebui să apară în ordinea în care este specificată. Valoarea din oficiu este `false`.

### `filter`

Este un obiect opțional de *interval filter*.

## Regula `any_of`

Regula returnează intervale generate de oricare dintre regulile interne.

### `intervals`

Este necesar să introduci un array de reguli care se combină. Un document trebuie să satisfacă toate regulile pentru a fi considerat.

### `filter`

Este un obiect opțional de *interval filter*.

## Regula `filter`

Această regulă returnează intervale în baza unui query.

```json
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "hot porridge",
          "max_gaps" : 10,
          "filter" : {
            "not_containing" : {
              "match" : {
                "query" : "salty"
              }
            }
          }
        }
      }
    }
  }
}
```

### `after`

Este un obiect de interogare opțional. Este folosit pentru a aduce intervale de documente aflate după intervalul principal al regulei `filter`.

### `before`

Este un obiect de interogare opțional. Este folosit pentru a aduce intervale de documente aflate înaintea intervalului principal al regulei `filter`.

### `contained_by`

Este un obiect de interogare opțional. Returnează intervale care se află în intervalele unei reguli `filter`.

### `containing`

Este un obiect de interogare opțional. Returnează intervale care conțin alte intervale ale unei reguli `filter`.

### `not_contained_by`

Este un obiect de interogare opțional. Returnează intervale care nu sunt conținute de vreun interval al unei reguli `filter`.

### `not_containing`

Este un obiect de interogare opțional. Returnează intervale care nu conțin un interval al unei reguli `filter`.

### `not_overlapping`

Este un obiect de interogare opțional. Returnează intervale care nu se suprapun cu intervale definite al unei reguli `filter`.

### `overlapping`

Este un obiect de interogare opțional. Returnează intervale care se suprapun cu intervale definite al unei reguli `filter`.

### `script`

Este un obiect opțional care retunează o valoare boolean.

## Script filters

Este folosit pentru a filtra intervale în baza poziției de start și a celei de final, fiind luate în considerare gap count-ul.

```json
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "hot porridge",
          "filter" : {
            "script" : {
              "source" : "interval.start > 10 && interval.end < 20 && interval.gaps == 0"
            }
          }
        }
      }
    }
  }
}
```

## Minimizare
