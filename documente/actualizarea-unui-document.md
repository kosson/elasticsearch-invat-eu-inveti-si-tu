# Actualizarea unui document

În Elasticsearch toate documentele nu pot fi modificate ceea ce conduce la concluzia că pentru a modifica un document, de fapt va fi șters anteriorul și creat unul nou care va fi indexat la rândul lui. Pentru actualizarea documentelor se va folosi endpoint-ul `_update`.

## Actualizarea simplă

Această operațiune se face folosind verbul `POST` precum în exemplul

```yaml
POST /nume_idx/_update/string_id
{
  "doc": {
    "câmp": "noua valoare"
  }
}
```

Putem avea un posibil exemplu:

```yaml
POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "doc": {
    "autor": "Ginel Alexăndrel"
  }
}
```

cu un răspuns similar cu:

```json
{
  "_index" : "miscelanee",
  "_type" : "_doc",
  "_id" : "t34t5XABCQdaqr4RL8i1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

Dacă un câmp nu există deja în document, acesta poate fi adăugat. Este aceeași operațiune menționând un câmp nou cu valoarea sa:

```yaml
POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "doc": {
    "tags": ["trist","nemilos"]
  }
}
```

Rezultatul poate fi similar cu:

```json
{
  "_index" : "miscelanee",
  "_type" : "_doc",
  "_id" : "t34t5XABCQdaqr4RL8i1",
  "_version" : 3,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "titlu" : "Undeva, cândva",
    "autor" : "Ginel Alexăndrel",
    "tags" : [
      "trist",
      "nemilos"
    ]
  }
}
```

## Actualizare scriptată - scripted updates

În loc să descarci un document, să-i actualizezi valoarea și apoi să-l încarci la loc, mai simplu este să creezi un scripted update, care face exact acest lucru evitând orice pas intermediar.

### Incrementare / decrementare valori

```yaml
POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "script": {
    "source": "ctx._source.likes++"
  }
}
```

În câmpul source se află scriptul de actualizare. Câmpul care trebuie actualizat este referit prin `ctx._source.nume_câmp`, unde `ctx._source` este înregistrarea (ctx - contextul). Pentru a crește o valoare, vom pune drept postfix `++`. În caz contrar, vom pune `--`.

### Atribuire valoare

```yaml
POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "script": {
    "source": "ctx._source.likes = 25"
  }
}
```

### Definire parametri

Definirea parametrilor se face folosind cheia "params", iar în cazul de mai jos, vom actualiza prin scădere din valoarea like-urilor folosind operatorul `-=`.

```yaml
POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "script": {
    "source": "ctx._source.likes -= params.quantity",
    "params": {
      "quantity": 5
    }
  }
}
```

În cazul în care condiționezi actualizarea unui document în funcție de o anumită valoare, poți evita actualizarea prin setarea acțiunii cu valoarea `noop`, precum în:

```yaml
POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "script": {
    "source": """
      if (ctx._source.likes == 20) {
        ctx.op = 'noop'
      }
      ctx.__source.likes--;
    """
  }
}
```

La nevoie, poți chiar să ștergi condițional un document setând valoarea lui `ctx.op` la valoarea 'delete'.

### Actualizare multiple documente folosind un query

Să presupunem că avem cazul unui subset de documente, care dacă îndeplinesc un anumit criteriu sau dacă au fost selectate de utilizator în interfață, dorim să-și modifice o anumită valoare toate. De exemplu, să le incrementăm cu o unitate sau să le decrementăm cu o unitate.

```yaml
POST /miscelanee/_update_by_query
{
  "script": {
    "source": "ctx._source.likes--"
  },
  "query":{
    "match_all": {}
  }
}
```

În exemplul propuse, conform query-ului ales, adică `match_all`, toate documentele vor decrementa valoarea din câmpul `likes` cu o unitate.

Pentru un index cu doar două elemente am putea avea un răspuns similar cu următorul obiect:

```json
{
  "took" : 246,
  "timed_out" : false,
  "total" : 2,
  "updated" : 2,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

Ori de câte ori se face un update folosind `_update_by_query`, se face câte un snapshot al întregului index. Apoi se face o interogare a tuturor *replication shards* existente în grupul de replicare. Acolo unde se găsesc documentele replicate, se trimite un *bulk update* pentru a le actualiza.

Specificarea lui `"conflict":"proceed"` va conduce la finalizarea operațiunii chiar dacă sunt detectate erori în versiunile prezente în shard-uri.

## Definirea de parametri în query

Acest mecanism este foarte util atunci când query-ul este făcut dinamic de vreo aplicație. De exemplu, poți deduce dintr-o valoare cea pasată dinamic prin acest parametru. Pot fi pasați mai mulți parametri într-un obiect dedicat acestora.

```yaml
POST /produse/_update/12
{
  "script": {
    "source": "ctx._source.exemplare -= params.cantitate",
    "params": {
        "cantitate": 12
    }
  }
}
```

## Ignorarea condiționată a unor documente cu noop

```yaml
POST /produse/_update/12
{
  "script": {
    "source": """
      if (ctx._source.exemplare = 0) {
        ctx.op = 'noop';
      } else if (ctx._source.exemplare < params.cantitate) {
        ctx._source.exemplare = 0;
      }
      ctx._source.exemplare -= params.cantitate;
    """,
    "params": {
        "cantitate": 12
    }
  }
}
```

## Ștergerea condițională a unui document

În cazul de mai jos, în cazul în care numărul de exemplare a fost epuizat, documentul va fi șters.

```yaml
POST /produse/_update/12
{
  "script": {
    "source": """
      if (ctx._source.exemplare <= 1) {
        ctx.op = 'delete';
      }
      ctx._source.exemplare--;
    """,
    "params": {
        "cantitate": 12
    }
  }
}
```

## Controlul concurenței

Acest lucru se referă la faptul că un același document poate fi actualizat în același timp. Pentru a preveni problemele care ar putea să apară, toate documentele expun două proprietăți importante (*primary term* și *sequence number*):

- `_primary_term` și
- `_seq_no`

În baza acestor proprietăți poți face o cerere de actualizare a unui document câtă vreme acesta are aceleași valori. În caz contrar, înseamnă că a fost actualizat între timp de altcineva. Să luăm un exemplu simplu.

```yaml
POST /test/_doc/1
{
  "a": 10,
  "b": "ceva"
}

GET /test/_doc/1
```

Vom obține un obiect de răspun similar cu următorul obiect JSON:

```json
{
  "_index" : "test",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "a" : 10,
    "b" : "ceva"
  }
}
```

Dacă se dorește o actualizare a documentului, pentru a fi siguri că aceasta se va face pe obiectul documentului așa cum era la momentul în care am inițiat actualizarea, vom trimite actualizarea menționând drept parametri de query cele două proprietăți cheie.

```yaml
POST /test/_update/1?if_primary_term=1&if_seq_no=0
{
  "doc": {
      "a": 21
  }
}
```

Actualizarea s-a făcut cu succes și este primit ca răspuns următorul obiect.

```json
{
  "_index" : "test",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

O privire atentă la obiectul returnat de `GET /test/_doc/1`, relevă faptul că versiunea s-a modificat, precum și proprietățile `_sec_no` și `_primary_term`.

```json
{
  "_index" : "test",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "a" : 21,
    "b" : "ceva"
  }
}
```

Pentru a gestionarea o stare de eroare, se va construi un algoritm la nivel de aplicație, care să identifice starea de eroare, să aducă din nou obiectul documentului, să extragă valorile pentru cele două câmpuri și să trimită din nou o actualizare.
