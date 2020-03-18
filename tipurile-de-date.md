# Tipurile de date

Tipurile de date care pot fi prezente în Elasticsearch pot fi categorisite în patru părți importante:

- Core data types
- Complex data types
- Geo data types
- Specialized data types


## Text data type

Aceste câmpuri sunt folosite pentru a indexa fragmente de text așa cum sunt, de exemplu descrieri ale unui produs.

## Keyword Data Type

Aceste date sunt folosite pentru a structura datele. Astfel de date sunt adrese de email, cuvinte cheie, etc. Sunt folosite pentru a filtra și pentru a realiza agregări.

## Numeric Data Types

Aceste date sunt de tip numeric:
- float;
- integer,
- long;
- short;
- double;
- byte;
- half_float;
- scaled_float.

## Date data type

Acest tip este folosit pentru a păstra date.

## Boolean Type

Acestea sunt bine-cunoscutele valori `true` și `false`.

## Binare

Pot fi stocate și fișiere binare la nevoie, dar nu este o practică recomandabilă.

## Range Data Types

Sunt folosite pentru intervale numerice, de exemplu un interval între 20 și 34. Aceste intervale pot fi:

- integer_range;
- float_range;
- long_range;
- double_range;
- date_range.

Un exemplu ar fi `{"gte": 10, "lte": 20}`.

## Object Data Type

Acestea sunt obiecte care ajung în Elasticsearch ca obiecte JSON. Acestea pot conține alte obiecte sau array-uri.

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

## Array-uri

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

Soluția pentru acest comportament este folosirea de Nested Data Types care sunt versiuni specializate ale tipurilor de obiecte și permit array-urilor de obiecte să fie interogate independent.
Pentru a putea face căutarea va trebui să croiți query-urile pentru a suporta aceste obiecte imbricate.

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


































