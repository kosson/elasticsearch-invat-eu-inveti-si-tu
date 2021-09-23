# Tipurile de date

Tipurile de date care pot fi prezente în Elasticsearch pot fi organizate în patru părți importante:

- Tipuri de bază
- Tipuri complexe
- Geo date
- Tipuri de date specializate


## Tipuri de bază

### Text data type

Aceste câmpuri sunt folosite pentru a indexa fragmente de text așa cum sunt, de exemplu descrieri ale unui produs. Adu-ți aminte că atunci când se face introducerea documentelor într-un index fără a folosi un mapping particularizat, va fi folosit cel din oficiu, care va face mapping-ul în baza tipurilor de date găsite ca valori ale câmpurilor. În cazul câmpurilor text, mapping-ul din oficiu, desemnează a fi deopotrivă de tip `text`, cât și `keywork`.

```json
"type" : "text",
"fields" : {
	"keyword" : {
		"type" : "keyword",
		"ignore_above" : 256
	}
}
```

Tipurile text sunt folosite pentru a se putea face căutări full-text. Valoarea acestui câmp va trece printr-un analizor.

### Keyword Data Type

Aceste date sunt folosite pentru a structura datele. Astfel de date sunt adrese de email, cuvinte cheie, numele unei categorii, etc. În general sunt fragmente de text specializate și sunt folosite pentru a filtra și pentru a realiza agregări. Aceste date nu sunt trecute printr-un analizor.

Acest tip de date poate fi privit ca fiind optim pentru căutarea exactă a unui fragment de text - o căutare unu-la-unu am putea spune opusă a ceea ce se petrece în cazul căutării full-text care parcurge textul printr-un analizor și apoi token-urile generate sunt introduse într-un index.

Aceste valori sunt folosite penru a structura documente în seturi sau pentru a agrega date. Poți să le privești precum valorile unui vocabular controlat.

### Numeric Data Types

Aceste date sunt de tip numeric:

- float;
- integer,
- long;
- short;
- double;
- byte;
- half_float;
- scaled_float.

### Date data type

Acest tip este folosit pentru a păstra date calendaristice.

### Boolean Type

Acestea sunt bine-cunoscutele valori `true` și `false`.

### Binare

Pot fi stocate și fișiere binare la nevoie, dar nu este o practică recomandabilă.

### Range Data Types

Sunt folosite pentru intervale numerice, de exemplu un interval între 20 și 34. Aceste intervale pot fi:

- integer_range;
- float_range;
- long_range;
- double_range;
- date_range.

Un exemplu ar fi `{"gte": 10, "lte": 20}`.

## Tipuri complexe

### Object Data Type

Acestea sunt obiecte care ajung în Elasticsearch ca obiecte JSON. Acestea pot conține alte obiecte sau array-uri.

## Obiecte cu proprietăți obiecte

Datele obiectelor ajung în Elasticseach ca obiecte JSON.

Un exemplu ar fi un obiect JSON cu următoarele date:

```json
{
	"autor": {
		"nume":"Ion",
		"prenume":"Creangă"
	},
	"titlu":"Caprele Irinucăi"
}
```

În Elasticsearch, datele obiectului JSON vor fi aplatizate în următorul format:

```json
{
	"autor.nume":"Ion",
	"autor.prenume":"Creangă",
	"titlu":"Caprele Irinucăi"
}
```

În caz că este necesară parametrizarea explicită a unui [obiect](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/object.html#object) prin definirea unui mapping, acest lucru este posibil.

### Array-uri de obiecte

Acestea sunt pur și simplu valorile multiple care pot exista într-un câmp. Atunci când ai array-uri de array-uri, la final, în Elasticsearch vor fi aplatizate într-unul singur.

Nu poți avea array-uri de obiecte pentru că acestea vor fi aplatizate și astfel se vor pierde referințele. Privind la exemplul de mai jos, veți înțelege mai repede.

```json
{
	"autori": [
		{"nume":"Ion", "opere":10},
		{"nume":"Alina", "opere":7}
	]
}
```

va fi aplatizat în

```json
{
	"autori.nume": ["Ion", "Alina"],
	"autori.opere": [10, 7]
}
```

Soluția pentru acest comportament este folosirea de *Nested Data Types* care sunt versiuni specializate ale tipurilor de obiecte și permit ca array-urile de obiecte să fie interogate independent.

Pentru a putea face căutarea va trebui să croiți query-urile într-un mod aparte pentru a interoga aceste obiecte imbricate. Aceste query-uri sunt numite *nested query*, fiind aplicate pe fiecare obiect din array.

## Geopoint Data Types

Aceste date `geo_point` sunt folosite pentru a desfășura operațiuni legate de coordonatele geografice. Aceste date geografice pot fi mai specifice:

- `point`,
- `polygon`,
- `linestring`,
- `multipoint`,
- `multilinestring`,
- `multipoligon`,
- `geometrycollection`,
- `envelope`,
- `circle`.

## IP Data Type

Acest tip de date este folosit pentru a stoca adrese IPv4 și IPv6.

## Completition Data Types

Acest tip de date `completition` oferă posibilitatea de a implementa funcționalități de tipul *caută-în-timp-ce-scrie*. Acest lucru se face cu ajutor *suggestor*-ilor.

## Attachment Data Type

Acest tip de date este folosit pentru a permite căutarea în documente de tip PDF, PPT, RTF, etc. Pentru a putea folosi acest tip de date este nevoie de un **Ingest Attachement Processor Plugin**. Pentru a efectua procedurile prin care documentele ajung în Elasticseach, se folosește în subsidiar Apache Tika.

Pentru a instala pe o mașină Linux: `sudo bin/elasticsearch-plugin install ingest-attachment`.

## Date specializate

Aceste tipuri de date pot fi adrese, de regulă date care nu se încadrează strict în ce am prezentat deja.

## Array-uri

În Elasticsearch nu este nevoie să declari tipul de câmp într-un anumit fel pentru cazul în care acesta are drept valoare un [array](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/array.html).

Trebuie respectat ca toate valorile din array să fie de același tip. Array-urile de array-uri vor fi aplatizate. Nu trebuie configurat absolut nimic pentru a introduce array-uri.

În momentul în care se face adăugarea dinamică a unui array, tipul de date al primului element din array va da valoarea `type` pentru field. Array-urile cu date mixate nu sunt acceptate de Elasticseach. Valorile `null` din array-ul documentului, vor fi substituite cu valoarea setată prin parametrul de mapping `null_value`.

Pentru array-urile de obiecte, vezi [nested](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/nested.html).

Un exemplu care oferă claritate este cel al etichetelor aplicate unui document.

```javascript
PUT /book/_mapping
{
	"properties": {
		"nume": {"type": "text"},
		"autor": {
			"type": "text",
			"fields": {
				"keyword": {
					"type": "keyword"
				}
			}
		},
		{
			"etichete": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword"
					}
				}
			}
		}
	}
}
```

## Resurse

- [Field datatypes](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-types.html)
