# Despre Elasticsearch

Elasticsearch este o tehnologie folosită pentru a analiza date, fie acestea text, fie numerice. Elasticsearch este scris în Java și folosește la rândul său Apache Lucene. Adesea veți realiza că Elasticsearch este parte a unei suite software numite generic Elastic Stack. Componentele acestei suite sunt:

- Elastisearch,
- Logstash,
- Beats,
- Kibana (o platformă de analiză și vizualizare),
- X-Pack

Suita Elastic folosește un limbaj propriu utilizat la interogare numit *Query DSL* (Domain Specific Language). În esență acesta constă dintr-un obiect JSON care parametrează căutarea.

Elasticsearch este un *document store* distribuit. În loc de a stoca informație în râdurile unei tabele, Elastisearch stochează structuri de date complexe serializate ca documente JSON. Atunci când ai mai multe noduri Elasticsearch într-un cluster, documentele stocate sunt distribuite în cluster și pot fi accesate imediat din oricare dintre noduri.

Elasticsearch folosește o structură de date numită *index inversat* care oferă o regăsire foarte rapidă în cazul căutării full-text. Un index inversat introduce într-o listă fiecare cuvânt unic care apare în oricare document și identifică toate documentele în care respectivul cuvânt apare.

Un index poate fi privit ca o colecție optimizată de documente în care fiecare document este o colecție de câmpuri care sunt perechi cheie-valoare ce conțin datele. Datele fiecărui câmp vor fi indexate din oficiu. Fiecare câmp indexat are o structură de date optimizată dedicată.

Câmpurile text sunt stocate în indecși inversați, iar câmpurile numerice și cele geo sunt stocate în arbori BKD. Această caracteristică de folosire a datelor la nivel de câmp, dă rapiditatea în căutările Elastisearch.

Datele stau în ceea ce se numește **nod**. Aceste noduri permit o scalare orizontală. Mai multe noduri constituie un **cluster**. Clusterele sunt complet independente unele de celelalte. În Elastisearch datele sunt numite **documente**. Documentele sunt organizate în **indecși**, care au rolul de a grupa logic documentele similare. Un cluster va avea cel puțin un **index inversat**. Dacă ai un index care are trei câmpuri indexate la nivel de document, vei avea trei indexuri inversate. Indexul inversat constă din toți termenii din câmpurile indexate ale tuturor documentelor unui index.

Interogarea unei instanțe de Elasticsearch se face printr-un REST API. Datele de interogare sunt în format JSON.

## Descrierea componentelor

### Kibana

Este o poartă de acces către datele din Elasticsearch. Kibana permite crearea de așa-numite dashboards, care permit vizualizarea datelor.

### Logstash

Permite trimiterea în Elastisearch de loguri ale unor aplicații, cu scopul de a fi analizate. Mai mult de atât, poate fi considerat o adevărată uzină de prelucrare a datelor. Tot ce primește Logstash poate fi tratat ca evenimente. În funcție de tipul evenimentului, Logstash trimite date către o componentă serviciu care are capacitatea de a le prelucra (filters, inputs și outputs).

Inputurile pot fi fișiere care gestionează evenimente sau apeluri HTTP sau poate fi un câmp dintr-o bază de date care se modifică, ori un mesaj dintr-o listă a unei instanțe Kafka.

Filtrele se rezumă la felul în care sunt prelucrate datele. Poți pasa unei instanțe de Logstash un fișier CSV, XML sau JSON. Odată primit, un astfel de fișier cu date poate fi analizate pentru a fi îmbogățite, filtrate, pentru a se face o căutare într-o bază de date, etc. Folosind un plugin, se pot trimite datele în ceea ce se numește *stashes*. Un astfel de stash poate fi o instanță de Elastisearch, o listă Kafka, un email care este trimis automat, etc.

## X-Pack

Este un pachet care adaugă caracteristici funcționale suplimentare lui Elastisearch și Kibanei.

X-Pack adaugă posibilitatea de gestionare a securității pentru Elastisearch și Kibana. Tot cu acest pachet poți face o monitorizare a suitei Elastic. Ai acces astfel la date privind folosirea procesorului, a folosirii memoriei, a spațiului discului, etc. Poți seta chiar alerte, de exemplu atunci când memoria este ocupată în întregime sau procesorul este la capacitate maximă. Tot acest pachet oferă posibilitatea de a raporta anumite date, dar și de a descărca date în formate precum CSV.
Algoritmii de machine learning pe care îi folosește Elasticsearch, tot X-Pack îi pune la dispoziție.
X-Pack permite constituirea de forcast-uri pe baza cărora să poți aloca resurse de calcul sau să scalezi anumite acțiuni în funcție de evenimente, perioade ale anului, etc.

X-Pack mai pune la dispoziție posibilitatea de a analiza datele sub formă de grafuri și implicit este oferită posibilitatea de a analiza relațiile care s-au stabilit în datele pe baza căruia au fost generate graf-urile. În baza acestor grafuri poți face recomandări de acces pe alte resurse în baza relevanței și a similitudinii. În acest scenariu, Kibana poate oferi o vizualizare interactivă a graf-ului.
X-Pack oferă posibilitatea interogărilor formulate ca SQL - Structured Query Language.

## Beats

Sunt pachete software care trimit date către Elasticsearch. Acești adevărați „poștași”, au capacitatea de a trimite diferite tipuri de date:

- Filebeat trimite fișiere, posibil loguri de Apache, NGINX - access sau errors;
- Metricbeat trimite informații privind CPU, RAM, etc.
- Packetbeat colectează cereri HTTP sau tranzacții realizate între diferite baze de date;
- Winlogbeat colectează loguri specifice sistemului de operare Windows;
- Auditbeat colectează informații necesare auditării sistemelor LINUX/GNU;
- Hearbeat monitorizează prezența servicului.

## Resurse

- [Introduction to Apache Lucene & Elasticsearch | Ivaylo Pavlov | youtube | Jan 31, 2021](https://www.youtube.com/watch?v=BvgGgkN3clI)
- [Beginner's Crash Course to Elastic Stack - Part 1: Intro to Elasticsearch and Kibana | Official Elastic Community | Beginner's Crash Course to Elastic Stack | Official Elastic Community | youtube](https://www.youtube.com/watch?v=gS_nHTWZEJ8&list=PL_mJOmq4zsHZYAyK606y7wjQtC0aoE6Es)
