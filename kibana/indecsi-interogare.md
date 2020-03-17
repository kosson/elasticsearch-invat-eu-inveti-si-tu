# Interogarea indecșilor

Poți introduce o comandă de interogare de genul `GET /_cat/indices?v`. Această comandă aduce informații foarte valoroase privind fiecare indice existent. Informația poate fi similară cu următorul răspuns:

```text
health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .monitoring-kibana-7-2020.03.10 aTEjyoQcTkucbU5xM1cyWw   1   0       3029            0    765.9kb        765.9kb
green  open   .monitoring-kibana-7-2020.03.11 CFWDuRb3SUCCsqkVCVRRJw   1   0       2602            0    596.2kb        596.2kb
yellow open   europeana                       HpCYxq4kSmOgpD86Pl-Rmw   5   1          2            0    179.6kb        179.6kb
green  open   .monitoring-es-7-2020.03.16     KW5bhWG1QIOIraZ3-Df4Og   1   0     134848       294831     72.9mb         72.9mb
green  open   .monitoring-kibana-7-2020.03.12 YXRM6jwfRLCTg5Sx4d7WSw   1   0       5142            0      1.2mb          1.2mb
```
