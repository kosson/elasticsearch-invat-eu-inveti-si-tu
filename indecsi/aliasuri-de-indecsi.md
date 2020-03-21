# Aliasuri de indecși

Acest API permite să creezi un alt nume pentru un index existent sau multipli indecși existenți pentru a folosi aceste nume ca alternative în momentul în care se face o operațiune de indexare.

Aceste aliasuri sunt utile atunci când:

- ai nevoie să faci o reindexare cu 0 downtime;
- vrei să grupezi mai mulți indecși împreună;
- vrei să realizezi niște view-uri asupra unui set de documente.

În cazul simplu, folosește endpointul `_alias`. Pentru cazurile mai complicate există și endpointul `_aliases`.

```yaml
PUT /movies/_alias/moviesX
```

În exemplul de mai sus tocmai am creat un alias pentru un index care deja există. O interogare simplă `GET /moviesX/_search?pretty` va oferi drept rezultat documentele care deja există în `movies`.

În locul indexului pentru care tocmai am făcut un alias poate fi `*`, `_all`, un regexp sau identificatori separați prin virgulă. Această flexibilitate conduce la concluzia că poți face un alias pentru mai multe indexuri, chiar toate, dacă dorești.

Pentru a șterge un alias, folosești verbul `DELETE`

```yaml
DELETE /movies/_alias/moviesX
```

Pentru a verifica dacă un alias există folosești `HEAD`

```yaml
HEAD /movies/_alias/moviesX
```

Dacă aliasul indexului există, răspunsul va fi `200 - OK`, iar în caz contrar `404 - Not Found`.

API-ul `_aliases` permite operațiuni la nivel atomic. Mai jos avem un exemplu în care am creat câteva indexuri și aliasuri cu care să facem câteva operațiuni.

```yaml
PUT /cevadestersdupa
PUT /pufi
PUT /pufi/_alias/pufi1

POST /_aliases
{
  "actions":[
    {"add":{"index":"pufi", "alias":"pufi2"}},
    {"remove": {"index":"pufi", "alias":"pufi1"}},
    {"remove_index":{"index":"cevadestersdupa"}}
  ]
}

HEAD pufi
HEAD pufi2
```

Este recomandabil ca în producție să fie folosite aliasuri în loc de indecși direct din motiv că poate mai târziu vei avea nevoie să reindexezi. În cazul în care anumite câmpuri ale documentelor s-au modificat și ai nevoie să faci o reindexare.

Dacă ai un index, fă-i un alias pe care să-l folosești în producție de genul `nume_productie` pe care să-l folosești în locul indexului. Când apare nevoia, faci o reindexare a datelor din indexul vechi în indexul nou. Apoi, ștergi aliasul de pe indexul vechi și creezi un alias cu același nume pentru indexul nou. Astfel vei face trecerea de pe un index pe altul fără downtime.
