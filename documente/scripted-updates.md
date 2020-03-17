# Scripted updates

În loc să descarci un document, să-i actualizezi valoarea și apoi să-l încarci la loc, mai simplu este să creezi un scripted update, care face exact acest lucru evitând orice pas intermediar.

## Incrementare / decrementare valori

```yaml
POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "script": {
    "source": "ctx._source.likes++"
  }
}
```

În câmpul source se află scriptul de actualizare. Câmpul care trebuie actualizat este referit prin `ctx._source.nume_câmp`, unde `ctx._source` este înregistrarea (ctx - contextul). Pentru a crește o valoare, vom pune drept postfix `++`. În caz contrar, vom pune `--`.

## Atribuire

```yaml
POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "script": {
    "source": "ctx._source.likes = 25"
  }
}
```

## Definire parametri

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
