# Crearea de indecși

Cel mai simplu mod de a crea un index este să îl creezi cu un `PUT /nume_index`. În acest moment s-a creat un primary shard și un replica shard. Sintaxa prezentată se referă la Dev Tools-ul din Kibana. Poți folosi foarte bine și `curl`.

Acest index va avea starea *yellow* pentru simplu motiv că nu a avut un node căruia să-i trimită replica shard-ul. Din acest motiv, starea este yellow. Atunci când este creat un alt nod, se va aloca replica shard-urile și starea va fi green.

Pentru a verifica numărul și starea indecșilor creați, se va folosi comanda `GET /_cat/indices?v`, care va indica informații foarte utile privind alocarea shard-urilor primare, replicile, starea indicelui și chiar și numărul de documente dintr-un indice.

## Reguli de scriere a numelui indexului

Numele indexului se va scrie respectând următoarele reguli:

- doar caractere lowercase;
- nu poate include următoarele caractere: `\`,`/`,`*`,`?`,`"`,`<`,`>`,`|`, spațiu, virgule sau `#`;
- nu poate fi folosit caracterul `:`,
- nu poate să înceapă cu următoarele caractere: `-`,`_` sau `+`;
- nu poate fi `.` sau `..`

Numărul de bytes maxim alocat este de 255.

## Bune practici

Dacă vor exista multe documente într-un index, performanța de căutare va scădea. Dacă majoritatea interogărilor se centrează pe același câmp al unui index, poți grupa logic datele acelui câmp. Datele unui astfel de câmp pot reflecta categorii, clase, tipuri.

De exemplu, dacă ai indexuri separate pe care ai dori să le interoghezi unitar, soluția este crearea unui alias pentru cele patru și apoi îl interoghezi pe acesta. Să presupunem că avem indecșii `pufi1` și `pufi2`. Aceștia pot fi indecși sau aliasuri ale unor indecși. Pentru a face un alias pe care să performăm interogări, vom proceda la un scenariu similar exemplului de mai jos:

```yaml
PUT /pufi*/_alias/toti_pufii

GET /pufi*/_alias/toti_pufii
{
  "pufi1":{
    "aliases":{
      "toti_pufii":{}
    }
  },
  "pufi2":{
    "aliases":{
      "toti_pufii":{}
    }
  }
}
```

## Referințe

- [Create index API | Index APIs | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)
