# Înlocuirea documentelor

Acest lucru se face folosind verbul `PUT` precum în exemplul de mai jos:

```yaml
PUT /miscelanee/_doc/100
{
  "titlu": "Moș Teacă și examenele",
  "autor": "Anton Bacalbașa",
  "likes": 1,
  "tags": ["interesant","prostia"]
}
```

Atenție, o astfel de operațiune va avea drept efect înlocuirea pe de-antregul a documentului vizat.
