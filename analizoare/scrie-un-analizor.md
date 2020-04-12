# Construcția unui analizor

Un analizor este un pachet compus din următoarele blocuri constructive de nivel elementar:

- filtre la nivel de caractere
- tokenizatoare și
- filtre pe tokenuri.

![](img/analysis-chain.png)

Analizorii din oficiu includ deja aceste blocuri constructive pentru a răspunde adecvat diferitelor limbi și tipuri de text. Elasticsearch totuși expune aceste blocuri constructive pentru a putea fi combinate cu scopul de a defini noi analizori parametrați pentru a răspunde într-un anumit fel.

## Filtre la nivel de caractere

Un astfel de filtru primește textul sub formă de stream de caractere și îl [transformă](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-charfilters.html) prin adăugarea, eliminarea sau modificarea de caractere. Un exemplu ar fi transformarea numeralelor arabo-hunduse (٠‎١٢٣٤٥٦٧٨‎٩‎) în versiunea Arabic-Latin (0123456789), sau eliminarea tagurilor HTML din conținut.

Un analizor poate avea zero sau mai multe filtre la nivel de caractere.

## Filtre la nivel de token

Un tokenizator, primește un stream de caractere pe care îl sparge în tokeni (de regulă la nivel de cuvânt) în momentul în care detectează un spațiu.
Tokenizerul este responsabil și cu ținerea minte a ordinii, adică a poziției fiecăruia dintre termeni, precum și începutul, dar și finalul la nivel de caractere care marchează limitele acelui cuvânt.

Un analizor are un singur tokenizator.

## Filtre la nivel de token

Un analizor poate avea unul sau mai multe filtre la nivel de token. Acestea primesc stream-ul de tokeni și adaugă, modifică sau elimină dintre ei. De exemplu, un filtru [lowercase](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-lowercase-tokenfilter.html) va transforma cuvintele în lowercase, un filtru [stop](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-stop-tokenfilter.html) va elimina cuvintele considerate *stop words*, iar un filtru [synonym](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-synonym-tokenfilter.html) va permite introducerea de sinonime.

## Resurse

- [Built-in analyzer reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-analyzers.html)
- [Anatomy of an analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analyzer-anatomy.html)
- [Writing analyzers](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/writing-analyzers.html)
- [Create a custom analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-custom-analyzer.html)
- [Text analysis](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis.html)
- [Tokenizer reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-tokenizers.html)
