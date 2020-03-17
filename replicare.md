# Replicarea în Elasticsearch

Replicarea este configurată la nivel de index. Replicarea constă în copierea shard-urilor pe care un index le conține. Aceste shard-uri replicate sunt numite *replica shards*. Acestea sunt copii ale unui shard și pot fi interogate la fel cum sunt interogate *primary shard*-urile. Numărul de replici poate fi configurat la momentul creării indexului. Valoarea din oficiu este unu. Pentru a preveni pierderea de date, *replica shard*-urile nu sunt ținute pe același nod cu *primary shard*-ul.
Un shard replicat este numit *primary shard*.
Un *primary shard* și ale sale *replica shards* sunt numite *replication group*.

În cazul în care avem două shard-uri pe același nod, replicile nu vor fi asignate pentru că avem un singur nod care rulează. Pentru atribuire, ar fi nevoie de un nod suplimentar pe care să le trimită. Fii avertizat de faptul că un nod suplimentar trebuie pus pe o altă mașină. Abia în momentul în care ai alt nod, abia atunci se vor repartiza și replicile.

Pe sistemele de producție este nevoie de cel puțin două noduri pentru a evita pierderile de date.

*Replica shard* -urile sunt folosite și pentru a elimina bottleneck-urile atunci când primary nu mai face față. Nu uita că un replica shar este o copie funcțională a unui shard, ceea ce înseamnă că pot fi interogate în același timp.

Dacă vei crea un index nou, dar disponibil este un singur nod, replica shard-ul care a fost creat odată cu primary-ul, nu are unde să fie alocat, ceea ce va seta *health*-ul indexului la *yellow*. Poți investiga starea shard-urilor cu `GET /_cat/shards?v`. În cazul nealocării replicii unui nod suplimentar, vei avea un rezultat de următorul tip:

```text
resursedus                      0     p      STARTED        29  75.2kb 127.0.0.1 nume_comp
resursedus                      0     r      UNASSIGNED
```

În cazul în care numărul de noduri crește, replicile vor fi alocate automat.

Atenție, creșterea numărului de shard-uri vine la pachet cu necesitatea creării unui nou index în care punem documentele și apoi facem reindexarea lor.
