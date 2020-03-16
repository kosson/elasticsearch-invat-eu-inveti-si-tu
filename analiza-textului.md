# Analiza textului

Analiza de text constă în convertirea textului în tokeni sau termeni care sunt adăugate în indexul inversat pentru căutare.
Analiza este făcută de un analizor care poate fi unui intern built-in sau poate fi unui construit per fiecare index.

În momentul în care se face tokenizarea, cuvintele vor fi setate la lowercase. Vor fi eliminate cuvintele frecvente iar celelalte sunt reduse la stem-uri.
De exemplu, pentru fragmentul de text `"The QUICK brown foxes jumped over the lazy dog!"`, vor fi generate stem-urile (foxes → fox, jumped → jump, lazy → lazi), iar la final, următoarele fragemente vor fi introduse în indexul inversat: `[ quick, brown, fox, jump, over, lazi, dog ]`.
