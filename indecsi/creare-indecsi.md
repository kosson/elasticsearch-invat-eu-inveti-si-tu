# Crearea de indecși

Cel mai simplu mod de a crea un index este să îl creezi cu un `POST /nume_index`. În acest moment s-a creat un primary shard și un replica shard.

Acest index va avea starea *yellow* pentru simplu motiv că nu a avut un node căruia să-i trimită replica shard-ul. Din acest motiv, starea este yellow. Atunci când este creat un alt nod, se va aloca replica shard-urile și starea va fi green.

Pentru a verifica numărul și starea indecșilor creați, se va folosi comanda `GET /_cat/indices?v`, care va indica informații foarte utile privind alocarea shard-urlor primare, replicile, starea indicelui și chiar și numărul de documente dintr-un indice.
