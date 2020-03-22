# Calculul relevanței

Până de curând, Elastisearch a folosit un algoritm de calcul al relevanței numit TF/IDF (Term Frequency/Inverse Document Frequency). În acest moment se folosește Okapi BM25.

Să inventariem factorii în baza cărora se calculează scorul de relevanță.

## Term Frequency

Calculează de câte ori apare un termen într-un câmp al unui document. Dacă termenul apare de mai multe ori, scorul de relevanță crește.

## Inverse Document Frequency

Algoritmul caută cât de des apare un termen în index, adică în toate documentele. Cu cât apare mai des un cuvânt, cu atât mai puțin relevant este acesta.

## Field-length norm

Cu cât un câmp este mai mare, cu atât apariția frecventă a unui cuvânt în acel câmp îl va face mai relevant. Un termen care apare într-un câmp redus ca dimensiune, are o relevanță mai mare.

## Explicarea calculului privind relevanța

Pentru a interoga Elasticsearch asupra felului în care a realizat calculul relevanței, în Dev Console (Kinbana) sau curl, dacă folosiți Terminalul, vom adăuga parametrul `explain`. În cazul adăugării în corpul interogării, setează `explain` la valoarea `true`.

```yaml
GET /movies/_search
{
  "query": {
    "term": {
      "title": "star"
    }
  },
  "explain": true
}
```

Una din concluziile foarte importante este aceea că scorul de relevanță este calculat în funcție de câte documente, în care a fost găsit termenul, se află pe un anumit *shard*. Importanța scade dacă sunt documente mai puține într-un shard.

Vom avea un răspuns la interogarea de mai sus în care fiecare rezultat va fi explicat din punct de vedere al modului în care a fost calculată relevanța.

Putem cere o explicație privind stabilirea relevanței la nivel de document (căutare după id), dacă acest lucru este dorit pentru a înțelege mai bine motivele pentru care un anumit document a fost găsit sau nu.

```yaml
GET /movies/_doc/122886
{
  "query": {
    "term": {
      "title": "star"
    }
  },
  "explain": true
}
```

Vei primi un răspuns similar cu obiectul următor:

```json
{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "122886",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 2,
  "found" : true,
  "_source" : {
    "id" : "122886",
    "title" : "Star Wars: Episode VII - The Force Awakens",
    "year" : 2015,
    "genre" : [
      "Action",
      "Adventure",
      "Fantasy",
      "Sci-Fi",
      "IMAX"
    ]
  }
}
```
