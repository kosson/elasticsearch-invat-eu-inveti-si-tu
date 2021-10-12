# Client helpers

Clientul de NodeJS oferă o colecție de funcționalități utile (*helpers*) pentru a oferi o experiență de lucru mai confortabilă cu unele API-uri Elasticseach.

Aceste funcționalități sunt în stadiu experimental și astfel, supuse modificărilor în releas-urile minore. Aceste funționalități au nevoie de o versiune NodeJS mai mare de 10.

## Bulk helper

Această funcționalitate oferă posibilitatea de a prelucra date de mari dimensiuni prin lucrul conjugat cu stream-urile.

```javascript
const { createReadStream } = require('fs');
const split = require('split2');
const { Client } = require('@elastic/elasticsearch');

const client = new Client({ node: 'http://localhost:9200' });

async function indexare () {
  const datasetPath = path.join(_dirname, '..', 'datelucru', 'settwitter.ndjson');
  const datasource = fs.createReadStream(datasetPath);

  const result = await client.helpers.bulk({
    datasource,
    // sau
    // datasource: createReadStream('./dataset.ndjson').pipe(split()),
    onDocument (doc) {
      return {
        index: { _index: 'twitter' }
      };
    },
    onDrop (doc) {
      console.log(`Nu am putut indexa `, doc);
    }
  });
}

indexare().catch((error) => {
  console.log(error);
});

console.log(result);
// {
//   total: number,
//   failed: number,
//   retry: number,
//   successful: number,
//   time: number,
//   bytes: number,
//   aborted: boolean
// }
```

### datasource

Poate fi un array cu date (stringuri sau obiecte JavaScript), un async generator sau un stream readable. Acestea sunt datele pe care vei putea face operațiuni CRUD. În cazul utilizării stream-urilor, se recomandă utilizarea pachetului `split2` care detectează și folosește delimitatorii din datele primite pentru a segmenta.

Pentru un array a cărui elemente sunt valori string, aplici direct `split()`, precum în exemplul de mai jos.

```javascript
datasource: createReadStream('./dataset.ndjson').pipe(split());
```

Dacă datele prelucrate sunt într-un array al cărui elemente sunt obiecte (JSON), îi pasezi lui `split()` un `JSON.parse`, precum în exemplul de mai jos.

```javascript
datasource: createReadStream('./dataset.ndjson').pipe(split(JSON.parse));
```

### onDocument

Aceasta este o funcție care se aplică pe fiecare document parcurs. În această funcție puteți prelucra datele documentului, iar ceea ce trebuie returnat este operațiunea pe care vrei să o desfășori cu acel document.
Conform documentației privind operațiunile `bulk` din [ghid](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-bulk.html), operațiunile posibile sunt cele din exemplul următor.

```javascript
const response = await client.bulk({
  body: [
    {
      index: {
        _index: 'test',
        _id: '1'
      }
    },
    {
      field1: 'value1'
    },
    {
      delete: {
        _index: 'test',
        _id: '2'
      }
    },
    {
      create: {
        _index: 'test',
        _id: '3'
      }
    },
    {
      field1: 'value3'
    },
    {
      update: {
        _id: '1',
        _index: 'test'
      }
    },
    {
      doc: {
        field2: 'value2'
      }
    }
  ]
});
console.log(response);
```

Câteva mențiuni importante în formatarea datelor le găsești tot în ghidul *Bulk API*.

### onDrop

Este o funcție care este apelată ori de câte ori un document nu poate fi indexat și s-a atins numărul maxim de încercări.

### flushBytes

Este dimensiunea maximă în bytes a corpului înainte de a fi trimis. Valoarea din oficiu este 5MB.

```javascript
const b = client.helpers.bulk({
  flushBytes: 1000000
});
```

### flushInterval

Este timpul cât așteaptă helperul (în milisecunde) înainte să curețe datele documentului anterior care a fost prelucrat. Valoarea din oficiu este 30000.

```javascript
const b = client.helpers.bulk({
  flushInterval: 30000
});
```

### concurrency

Indică numărul de cereri executate concomitent. Valoarea din oficiu este 5.

```javascript
const b = client.helpers.bulk({
  concurrency: 10
});
```

### retries

Indică numărul încercărilor înainte de a apela funcția `onDrop()`. Valoarea din oficiu este cea care este specificată de client pentru *max retries*.

```javascript
const b = client.helpers.bulk({
  retries: 3
});
```

### wait

Cât timp (milisecunde) să aștepte înainte de a încerca din nou. Valoarea din oficiu este 5000.

```javascript
const b = client.helpers.bulk({
  wait: 3000
});
```

### refreshOnCompletion

Dacă această opțiune are setată valoarea `true` la finalul operațiunii *bulk* va rula o operațiune *refresh* pe toți indecșii specificați. Valoarea din oficiu este `false`.

```javascript
const b = client.helpers.bulk({
  refreshOnCompletion: true
  // or
  refreshOnCompletion: 'index-name'
});
```

### Operațiunile posibile

#### Index

```javascript
client.helpers.bulk({
  datasource: myDatasource,
  onDocument (doc) {
    return {
      index: { _index: 'my-index' }
    }
  }
});
```

#### Create

```javascript
client.helpers.bulk({
  datasource: myDatasource,
  onDocument (doc) {
    return {
      create: { _index: 'my-index', _id: doc.id }
    }
  }
});
```

#### Update

```javascript
client.helpers.bulk({
  datasource: myDatasource,
  onDocument (doc) {
    // Note that the update operation requires you to return
    // an array, where the first element is the action, while
    // the second are the document option
    return [
      { update: { _index: 'my-index', _id: doc.id } },
      { doc_as_upsert: true }
    ]
  }
});
```

#### Delete

```javascript
client.helpers.bulk({
  datasource: myDatasource,
  onDocument (doc) {
    return {
      delete: { _index: 'my-index', _id: doc.id }
    }
  }
});
```

### Întreruperea unei operațiuni bulk

În caz că este necesar, poți opri operațiunea **bulk** în orice moment. Helperul *bulk* oferă posibilitatea prelucrării cu `then`, care pune la dispoziție o metodă `abort`.

```javascript
const { createReadStream } = require('fs');
const split = require('split2');
const { Client } = require('@elastic/elasticsearch');

const client = new Client({ node: 'http://localhost:9200' });
const b = client.helpers.bulk({
  datasource: createReadStream('./dataset.ndjson').pipe(split()),
  onDocument (doc) {
    return {
      index: { _index: 'my-index' }
    };
  },
  onDrop (doc) {
    b.abort();
  }
})

console.log(await b);
```

### Pasarea altor opțiuni ale API-ului bulk

Poți pasa oricare [opțiuni ale API-ului](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-bulk.html#docs-bulk-api-query-params) *bulk*.

```javascript
const result = await client.helpers.bulk({
  datasource: [...],
  onDocument (doc) {
    return {
      index: { _index: 'my-index' }
    }
  },
  pipeline: 'my-pipeline'
});
```

### Folosirea unui async generator

```javascript
const { Client } = require('@elastic/elasticsearch');

async function * generator () {
  const dataset = [
    { user: 'jon', age: 23 },
    { user: 'arya', age: 18 },
    { user: 'tyrion', age: 39 }
  ];
  for (const doc of dataset) {
    yield doc
  }
}

const client = new Client({ node: 'http://localhost:9200' });
const result = await client.helpers.bulk({
  datasource: generator(),
  onDocument (doc) {
    return {
      index: { _index: 'my-index' }
    }
  }
});

console.log(result);
```

## Helperul multi search

În cazul în care trimiți cereri de căutare într-o cadență rapidă, acest helper se dovedește a fi foarte util. În subsidiar folosește API-ul `multi search`. Rezultatul expune o proprietate numită `documents` în care găsești direct rezultatele.

```javascript
const { Client } = require('@elastic/elasticsearch');

const client = new Client({ node: 'http://localhost:9200' });
const m = client.helpers.msearch();

// API sub formă de promisiune
m.search(
    { index: 'stackoverflow' },
    { query: { match: { title: 'javascript' } } }
  )
  .then(result => console.log(result.body)) // sau result.documents
  .catch(err => console.error(err));

// API ce folosește callback-urile
m.search(
  { index: 'stackoverflow' },
  { query: { match: { title: 'ruby' } } },
  (err, result) => {
    if (err) console.error(err)
    console.log(result.body)); // sau result.documents
  }
);
```

### operations

Indică câte operațiuni ar trebui să fie trimise într-o singură cerere `msearch`. Valoarea din oficiu este 5.

```javascript
const m = client.helpers.msearch({
  operations: 10
});
```

### flushInterval

În cât timp (milisecunde) se va face ștergerea operațiunilor anterioare ale unui read. Valoarea din oficiu este 500.

```javascript
const m = client.helpers.msearch({
  flushInterval: 500
});
```

### concurrency

Câte cereri sunt executate în același timp. Valoarea din oficiu este 5.

```javascript
const m = client.helpers.msearch({
  concurrency: 10
});
```

### retries

De câte ori este reluată o operațiune pentru a rezolva o cerere. Valoarea din oficiu este cea care este specificată de client pentru *max retries*. O operațiune este reluată doar dacă ai o eroare 429 (429 Too Many Requests).

```javascript
const m = client.helpers.msearch({
  retries: 3
});
```

### wait

Cât să aștepte înainte de a încerca din nou. Valoarea din oficiu este 5000.

```javascript
const m = client.helpers.msearch({
  wait: 3000
});
```

### Cum oprești msearch helperul

Ai la îndemână o metodă `stop()`.

```javascript
const { Client } = require('@elastic/elasticsearch');

const client = new Client({ node: 'http://localhost:9200' });
const m = client.helpers.msearch();

m.search(
    { index: 'stackoverflow' },
    { query: { match: { title: 'javascript' } } }
  )
  .then(result => console.log(result.body))
  .catch(err => console.error(err));

m.search(
    { index: 'stackoverflow' },
    { query: { match: { title: 'ruby' } } }
  )
  .then(result => console.log(result.body))
  .catch(err => console.error(err));

setImmediate(() => m.stop());
```

## Helperul search

Este doar un wrapper peste API-ul de căutare (`search`). În loc să returneze întregul obiect `result`, va returna doar documentele. Pentru a îmbunătăți performanțele, acest helper adaugă automat `filter_path=hits.hits._source` la stringul de interogare.

```javascript
const documents = await client.helpers.search({
  index: 'stackoverflow',
  body: {
    query: {
      match: {
        title: 'javascript'
      }
    }
  }
})

for (const doc of documents) {
  console.log(doc)
}
```

## Helperul scroll search

În momentul apelării este returnat un *async iterator* care poate fi prelucrat cu un for...await...of. Gestionează automat eroarea 429 și folosește opțiunea `maxRetries` a clientului.

```javascript
const scrollSearch = client.helpers.scrollSearch({
  index: 'stackoverflow',
  body: {
    query: {
      match: {
        title: 'javascript'
      }
    }
  }
});

for await (const result of scrollSearch) {
  console.log(result);
}
```

### Clear scroll search

În caz că este nevoie, poți curăța/elimina/întrerupe o căutare scroll apelând `result.clear()`.

```javascript
for await (const result of scrollSearch) {
  if (condition) {
    await result.clear();
  }
}
```

### Obținerea documentelor

Când ai nevoie de documentele unui scoll search, le poți obține prin `result.documents`.

```javascript
for await (const result of scrollSearch) {
  console.log(result.documents)
}
```

## Helperul scoll documents

Acest helper lucrează într-o manieră similară helperului scroll search, dar returnează doar documentele. Fiecare loop returnează un singur document. Nu poți folosi metoda `clear`. Pentru a îmbunătăți performanțele, acest helper adaugă automat `filter_path=hits.hits._source` la stringul de interogare.

```javascript
const scrollSearch = client.helpers.scrollDocuments({
  index: 'stackoverflow',
  body: {
    query: {
      match: {
        title: 'javascript'
      }
    }
  }
});

for await (const doc of scrollSearch) {
  console.log(doc);
}
```

## Resurse

- [Client helpers](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/master/client-helpers.html)
- [Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-bulk.html)
- [Elasticsearch Node.js work-with-me and AMA | Tomas Della Vedova | Streamed live on Aug 19, 2021](https://www.youtube.com/watch?v=Jk4_4k1N3yw)
