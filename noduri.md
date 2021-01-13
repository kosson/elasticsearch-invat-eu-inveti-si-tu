# Noduri

Regula de aur spune să ai câte un nod per mașină. Pentru sistemele de producție este nevoie de cel puțin două noduri.

## Roluri

Nodurile pot îndeplini mai multe roluri

### Rolul *master*

Acest nod are responsabilități stabilite la nivel de cluster. Acest lucru îi permite crearea și ștergerea de indecși, ținerea evideței nodurilor și a shardurilor. Alegerea nodului master se face printr-un așa-numit vot. În cazul sistemelor de producție, având clustere de mari dimensiuni, este un lucru de dorit să ai un master node dedicat.

### Rolul de *data*

Acest rol îi permite să stocheze date, ceea ce permite performarea de query-uri legate de acele date, precum sunt query-urile de căutare.

### Rolul de *ingest*

Din acest rol, un nod permite acomodarea unor pipeline-uri de ingest. Un astfel de pipeline sunt pași performați la momentul în care sunt indexate documente. Acest lucru permite manipularea documentelor, folosind ceea ce se cheamă *processors* într-un pipeline.

Poți compara acest rol cu funcționalitatea, ce-i drept redusă a lui Logstash.

### Rol de *machine learning*

Orice nod identificat cu *node.ml* este unul dedicat machine learning-ului. Acest nod va rula job-uri de machine learning. Este în tandem cu `xpack.ml`.

### Rol *coordination*

Poate fi folosit ca load balancers în clustere.

Dacă elimini toate rolurile unui nod, singurul rămas este cel de coordonare:

- node.master: false
- node.data: false
- node.ingest: false
- node.ml: false
- xpack.ml.enabled: false


### Rol *voting-only*

Este un nod care participă în procesul de votare a nodului, care va avea rol de master. Acest nod nu va putea fi el însuși master-ul.
