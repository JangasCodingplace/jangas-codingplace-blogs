# Spaß mit Zeitreihen - ein Überblick
Werden Daten in Abhängigkeit von der Zeit repräsentiert, handelt es sich um eine Zeitreihe. Oder auch in anderen Worten: eine Zeitreihe ist eine sequenzielle Abfolge von Daten.

Typische Anwendungsfälle der Zeitreihenanalyse sind:
- Statistische Auswertung von Zeitreihen
- Trenderkennung und sequenzielle Vorhersagen von Daten
- Anomalieerkennung

In diesem Beitrag wird sich mit verschiedenen Aspekten sowie konkreten Beispielen der Zeitreihenanalyse beschäftigt. Darunter folgendes:
- Möglichkeiten zum persistieren von Zeitreihen
- descriptive Analyse von Zeitreihen mit SQL
- Vorhersage von statischen Zeitreihen am Beispiel einer Kapazitätsplanung
- Vorhersage von veränderlichen Zeitreihen am Beispiel eines Ticketverkaufs
- Deep Learning
- Anomalieerkennung

## Speichern von Zeitreihen
Es gibt eine Vielzahl an Möglichkeiten Zeitreihen zu persistieren. Allem voran die gute alte CSV Datei. Aber auch verschachtelte Informationen als JSON Dokumente sind durchaus denkbar.

Entsprechend eignen sich auch viele gängige SQL-Datenbanken wie MySQL, PostgreSQL, OracleDB, MariaDB sowie NoSQL Datenbanken wie MongoDB oder auch Elasticsearch sehr gut zum ablegen von Zeitreihen.

Daneben gibt es aber auch Datenbanken speziell für Zeitreihen, z.B. TimeScaleDB, InfluxDB oder Prometheus um mal ein paar zu nennen.

Am Ende muss die Frage gestellt werden, mit welchen Daten gearbeitet wird und ob speziell von bestimmten Zeitreihendatenbanken profitiert wird.
So kann es sein, dass auf einem statischen Datensatz gearbeitet wird, welcher nicht weiter wächst. Normale Dokumente können hier bereits ausreichen.
Es kann auch sein, dass es bereits eine Datenbank für Stammdaten gibt und dass die Zeitreihen nicht zu schnell wachsen. In dem Fall kann die bereits verwendete Datenbank total ausreichend sein.
Es kann aber auch sein, dass die mitgelieferten Features von Zeitreihendatenbanken einen großen Bonus darstellen. So gibt es in derartigen Datenbanken oft bestimmte Retention Policies welche Daten ab einem bestimmten Zeitraum automatisch archivieren, wegwerfen oder zusammenfassen. Oder auch schlicht bestimmte Features in den entsprechenden Querylanguages.

## Methoden zur Zeitreihenanalyse & Vorhersage von Zeitreihen
Es gibt eine Vielzahl verschiedener Methoden. Die Zeitreihenanalyse ist ein riesiges Feld für das ganze Bücher geschrieben werden. Unabhängig der hier aufgeführten Methoden macht ein grundsätzliches Verständnis in den Bereichen Statistical Learning als Grundlage absolut Sinn.

Ein Buch welches ich vor Jahren empfohlen bekommen habe ist der etwa 600 Seiten lange, aber frei verfügbare Wälzer [An Introduction into Statistical Learning](https://www.statlearning.com/). Ich finde ein gutes Nachschlagewerk mit einer netten Übersicht zu den verschiedenen Methoden.

Solltest du über Deep Learning Methoden in der Zeitreihenanalyse nachdenken, machen natürlich auch Grundlagen in eben diesem Bereich Sinn, bevor du einsteigst.

In erster Linie sind die Grundlagen aber auch dafür wichtig zu lernen, wie Modelle richtig bewertet werden können und wie Daten explorativ aufgearbeitet werden. In vielen Beiträgen speziell zum Thema Zeitreihen - so auch hier - werden wesentliche Bestandteile der Datenaufarbeitung, ihrer Interpretation sowie Interpretation und Diskussion der Ergebnisse vernachlässigt.

Da es unzählige Tools und Methoden um das Thema Zeitreihenanalyse gibt, hier ein kleiner Ausschnitt:
- SQL, Zeitreihendatenbanken mit entsprechenden built-in Funktionen, Python Pandas
- WMA (Weighted Moving Average), EWMA (Exponentially Moving Average)
- Holt-Winters-Method
- Autocorrelation Analysis (ACF, PACF)
- Autoregressive Integrated Moving Average (AMIRA, SAMIRA)
- Vector Auto Regression (VAR)
- Long Short-Term Memory (LSTM)
- Facebook Prophet Library

Wie gesagt. Nur ein Ausschnitt - lieblos hingeworfene Keywords. In diesem Beitrag gehe ich auf wenige von ihnen ein.

## Beispiel 1: E-Commerce Daten & SQL
Im folgenden bediene ich mich einmal am erstbesten E-Commerce Datenset auf Kaggle welches ich gefunden habe: [E-Commerce Data](https://www.kaggle.com/datasets/carrie1/ecommerce-data)

Das konkrete Projekt findest du bei mir auf GitHub unter: https://github.com/JangasCodingplace/e-commerce-sql-timeseries

Zu diesem Datensatz stelle ich einfach folgende Fragen:
- Was ist das Datum des letzten Einkaufs eines jeden Users und wie hoch war sein Einkaufswert an diesem Tag?
- Gibt es User die seit 3 Monaten nichts mehr gekauft haben?
- Was ist der _Durchschnittliche_ Einkaufswert pro User pro Monat und was ist der einkaufsstärkste User pro Monat?
- gemessen am Vormonat: zu welchen Uhrzeiten sollte besonders viel Support-Personal verfügbar sein?
- wie ist der Trend der Verkäufe pro Monat?

Beachte bitte, dass der Datensatz schon ein paar Tage älter ist und nicht jünger wird. Entsprechend wird der letzte Monat in dem Datensatz als "aktueller Monat" angesehen.

Ehe die Kanone ausgepackt wird und mit Jupyter Notebook, Pandas und Python gearbeitet wird, würde ich hier gerne einmal sehen, welche Möglichkeiten eine SQL Datenbank zur Beantwortung dieser Fragen bildet.
Der Vorteil einer reinen Datenbanklösung ist, dass sie vermutlich ohnehin bereits im Einsatz ist und damit kein neues Tooling aufgesetzt werden muss. Eine derartige Lösung kann bequem in den Adminstrationsbereich eines E-Commerce Shops oder bestehende BI Lösungen integriert werden.
Zudem wird SQL in Zeiten von ORMs echt vernachlässigt und es ist nützlich mit SQL wirklich umgehen zu können. Ich war selbst lange Zeit primär Django Entwickler und spreche mich davon nicht frei. Aus purem Egoismus frische ich auf diese Weise also gerne mein SQL ein wenig auf.

### Aufsetzen der Datenbank
Für diesen Beitrag wird die Datenbank mit Docker und Docker-Compose betrieben. _In vielen Fällen ist ein Datenbank UI übrigens auch bereits in einen Editor wie VSCode oder einer IDE wie JetBrains PyCharm integriert. Hier genügt es oft nach entsprechenden Plugins zu suchen._ Im konkreten Fall gebe ich dir Postgres Admin als UI mit in die docker-compose. Ich selbst werde aber das SQL-Tools Plugin mit dem PostgreSQL driver von VSCode nutzen. Sowohl VSCode als auch die notwendigen Plugins sind gratis:
- [VSCode](https://code.visualstudio.com/)
- [SQL Tools Plugin](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)
- [PG Driver](https://marketplace.visualstudio.com/items?itemName=ms-ossdata.vscode-postgresql)

Ich persönlich arbeite für den Code ohnehin in VSCode. Daher bevorzuge ich VSCode als UI für die Datenbank. So habe ich ein Tool für alles. Es liegt aber bei dir ob du einen anderen Ansatz bevorzugst.

Jetzt aber endlich an das konkrete Projekt! Zuerst wird ein kleines `init.sql` Skript erstellt, welches eine Tabelle mit dem korrekten Schema erstellt und die CSV Daten direkt lädt.
```sql
CREATE TABLE IF NOT EXISTS sales (
    InvoiceNo INTEGER,
    StockCode VARCHAR,
    Description VARCHAR,
    Quantity INTEGER,
    InvoiceDate TIMESTAMP,
    UnitPrice NUMERIC,
    CustomerID INTEGER,
    Country VARCHAR
);

COPY sales(InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country)
FROM '/docker-entrypoint-initdb.d/data.csv'
WITH (FORMAT CSV, HEADER, ENCODING  'LATIN1');
```


Als nächstes folgt auch schon die `docker-compose.yaml` File. Hierbei ist zu beachten, dass das `init.sql` Skript sowie die CSV von Kaggle unbedingt ins richtige Verzeichnis gelegt wird. In meinem Fall ist es `pgdb/scripts/`.
Solltest du parallel bei mir auf GitHub schauen, beachte bitte, dass ich die CSV von Kaggle nicht im Repository habe. Diese musst du separat herunterladen und richtig platzieren.

```yaml
version: '3.9'

services:
  db:
    image: postgres:latest
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: ecommerce
    volumes:
      - ./pgdb/scripts/:/docker-entrypoint-initdb.d
      - pgdata:/var/lib/postgresql/data

  pg-admin:
    image: dpage/pgadmin4:latest
    ports:
      - "8080:80"
    environment:  
      PGADMIN_DEFAULT_EMAIL: admin@admin.com  
      PGADMIN_DEFAULT_PASSWORD: admin

volumes:
  pgdata:
```

### Was ist das Datum des letzten Einkaufs eines jeden Users und wie hoch war der Wert?
Die Frage kann in zwei Schritten beantwortet werden. Als erstes holen wir uns die Daten der letzten Käufe:
```sql
SELECT CustomerID, MAX(InvoiceDate::DATE) AS LastPurchaseDate
FROM sales
GROUP BY CustomerID;
```

Die Rückgabe:
| CustomerID | LastPurchaseDate |
| ---------- | ---------------- |
|            | 2011-12-09       |
| 16592      | 2011-12-05       |
| 13527      | 2011-11-06       |
| 14173      | 2011-11-29       |
| 12502      | 2011-09-05       |

Nun fehlen noch die Summierten Werte des jeweiligen Tages.

```sql
SELECT 
	CustomerID, 
	MAX(InvoiceDate::DATE) AS LastPurchaseDate, 
	SUM(UnitPrice * Quantity) AS TotalValueOnDate
FROM sales
WHERE (CustomerID, InvoiceDate::DATE) IN (
	SELECT CustomerID, MAX(InvoiceDate::DATE) AS LastPurchaseDate
	FROM sales
	GROUP BY CustomerID
)
GROUP BY CustomerID
ORDER BY LastPurchaseDate DESC;
``` 

| CustomerID | LastPurchaseDate | TotalValueOnDate |
| ---------- | ---------------- | ---------------- |
| 12526      | 2011-12-09       | 277.08           |
| 16705      | 2011-12-09       | 250.52           |
| 15311      | 2011-12-09       | 439.85           |
| 12662      | 2011-12-09       | 224.95           |
| 17315      | 2011-12-09       | -7.50            |

Beachte bitte, dass ich hier nur einen Ausschnitt der Daten angebe und nicht von "ganz oben" aus starte. Mir war es wichtig dir zu zeigen, dass es auch negative Werte gibt. In dem Fall `-7.50`. Das bedeutet, dass es am entsprechenden Tag einen Umtausch gegeben haben könnte.

Bezüglich der Query sollte zu erkennen sein, dass eine verschachtelte Abfrage vorliegt. Die innere Abfrage entspricht jeweils der zuerst eingeführten abfrage und gibt das Datum des letzten Einkaufs pro User an.
Die äußere Query schaut, welche Einkäufe am jeweils letzten Einkauf eines Users konkret getätigt wurden. Alternativ zum `WHERE IN` kann hier auch ein `INNER JOIN` verwendet werden. Das ist auf große und indizierte Daten womöglich um einiges performanter.

Cool finde ich die Möglichkeit eine Summe über das Produkt zweier Spalten zu bilden: `SUM(UnitPrice * Quantity) AS TotalValueOnDate`. Das ist ein Feature welches oft Pandas zugeschrieben wird und es wird gerne vergessen, dass sowas auch in SQL möglich ist und nichtmal Postgres exklusiv!
Auch das ganze Value Parsing finde ich persönlich deutlich angenehmer als in Pandas. Repliziere das Ergebnis gerne mal in einem Jupyter Notebook mit Python Pandas und mach' dir davon ein eigenes Bild.

### Gibt es User die seit 3 Monaten nichts mehr gekauft haben?
In Analysen muss man oft nicht den Weg wählen, der am performantesten ist, sondern den, den man am besten versteht. Gleiches gilt übrigens auch für das programmieren.

Diese Aufgabe lässt sich Wunderbar in 3 Teile unterteilen:
- finde User die im Zeitraum zwischen 3 und 4 Monaten etwas gekauft haben (Gruppe 1)
- finde User die in den vergangenen 3 Monaten etwas gekauft haben (Gruppe 2)
- identifiziere die Nutzer, die in Gruppe 1 sind, aber nicht in Gruppe 2

Fangen wir mit der ersten Gruppe an:
```sql
WITH max_date AS (
    SELECT MAX(InvoiceDate) AS max_invoice_date FROM sales
)
SELECT DISTINCT(CustomerID) FROM sales
WHERE (
    InvoiceDate BETWEEN 
        (SELECT max_invoice_date FROM max_date) - INTERVAL '4 months'  
        AND (SELECT max_invoice_date FROM max_date) - INTERVAL '3 months'
    );
```

Easy! Hier passiert nichts wildes. Etwas schade finde ich, dass es in Postgres etwas umständlich ist lokale Variablen zu Deklarieren. In anderen Datenbanksystemen geht das einfacher.

Die zweite Abfrage ist recht ähnlich:
```sql
WITH max_date AS (
    SELECT MAX(InvoiceDate) AS max_invoice_date FROM sales
)
SELECT DISTINCT(CustomerID) FROM sales
WHERE
    InvoiceDate > (SELECT max_invoice_date FROM max_date) - INTERVAL '3 months';
```

Nun können beide Abfragen zu einer zusammen gesetzt werden.

```sql
WITH max_date AS (
    SELECT MAX(InvoiceDate) AS max_invoice_date FROM sales
),
group1 AS (
    SELECT DISTINCT(CustomerID) FROM sales
    WHERE (
        InvoiceDate BETWEEN 
            (SELECT max_invoice_date FROM max_date) - INTERVAL '4 months'  
            AND (SELECT max_invoice_date FROM max_date) - INTERVAL '3 months'
        )
),
group2 AS (
    SELECT DISTINCT(CustomerID) FROM sales
    WHERE InvoiceDate > (SELECT max_invoice_date FROM max_date) - INTERVAL '3 months'
)
SELECT group1.CustomerID FROM group1 
LEFT JOIN group2 ON group1.CustomerID = group2.CustomerID
WHERE group2.CustomerID IS NULL;
```
Wie du siehst, wird hier mit dem `WITH` Statement gearbeitet. Theoretisch kannst du die Abfrage auch beliebig verschachteln und damit u.U. sogar verkürzen. Auch lässt sich drüber streiten, wo der Richtige Ort für den `DISTINCT` ist - oder wo er nicht ist.

Das Ergebnis müsste dann irgendwie so aussehen:

| CustomerID | 
| ---------- |
| 12363      |
| 12399      |
| 12418      |
| 12422      |
| 12436      |

Wie eingangs erwähnt: manchmal geht es einfach darum etwas leicht verständlich zu machen und nicht immer darum zu optimieren und ggf. zu überoptimieren. Eine Meinung die ich sogar in Interviews vertrete.

Übrigens ist die Überoptimierung ein Fehler den ich selbst schon viel zu oft gemacht habe und auch immer wieder selbst mache ... _aber wer nicht?!_

### Was ist der _durchschnittliche_ Einkaufswert pro User pro Monat und was ist der einkaufsstärkste User pro Monat?
Endlich etwas, was schnell geht! Zwei Teilaufgaben, zwei Datenbankabfragen. Let's Go!

In der ersten Teilaufgabe geht es um den durchschnittlichen Einkaufswert pro User pro Monat. Hinter dieser Frage steckt ein recht fieses `GROUP BY` Statement:
```sql
SELECT 
    CustomerID AS cid,
    InvoiceNo AS invoiceno,
    DATE_PART('year', InvoiceDate) AS year,
    DATE_PART('month', InvoiceDate) AS month,
    SUM(UnitPrice * Quantity) AS total_val_per_buy,
    AVG(UnitPrice * Quantity) AS avg_val_per_buy,
    COUNT(*) AS buy_count
FROM sales
GROUP BY 
    CustomerID,
    InvoiceNo,
    DATE_PART('year', InvoiceDate),
    DATE_PART('month', InvoiceDate);
```
Wichtig zu beachten ist, dass nicht einfach nach `CustomerID` gruppiert wird. In der Tabelle sind einzelne Artikelpositionen. Ein Einkauf wird also zusammengesetzt aus `CustomerID` und `InvoiceNo`. Werden diese gruppiert, ergibt das den Einkauf.

Die Rückgabe dieser Abfrage müsste irgendwie so aussehen:
| cid   | invoiceno | year | month | total_val_per_buy | avg_val_per_buy | buy_count |
| ----- | --------- | ---- | ----- | ----------------- | --------------- | --------- |
| 12346 | 541431    | 2011 | 1     | 77183.60          | 77183.60000000  | 1         |
| 12346 | C541433   | 2011 | 1     | -77183.60         | -77183.6000000  | 1         |
| 12347 | 537626    | 2010 | 12    | 711.79            | 22.96096774193  | 31        |
| 12347 | 542237    | 2011 | 1     | 475.39            | 16.39275862068  | 29        |
| 12347 | 549222    | 2011 | 4     | 636.25            | 26.51041666666  | 24        |
| 12347 | 556201    | 2011 | 6     | 382.52            | 21.25111111111  | 18        |

Der zweite Teil der Aufgabe ist nicht mehr ganz so einfach. Es soll pro Monat der Umsatzstärkste User identifiziert werden und alle anderen weg gelassen werden. Das ist der Punkt, an dem s.g. _window functions_ zum Einsatz kommen. Aber schau selbst:

```sql
SELECT CustomerID, PurchaseYear, PurchaseMonth, TotalPurchaseValue
FROM (
    SELECT 
        CustomerID,
        DATE_PART('year', InvoiceDate) AS PurchaseYear,
        DATE_PART('month', InvoiceDate) AS PurchaseMonth,
        SUM(UnitPrice * Quantity) AS TotalPurchaseValue,
        RANK() OVER (
            PARTITION BY 
                DATE_PART('year', InvoiceDate), 
                DATE_PART('month', InvoiceDate) 
                ORDER BY SUM(UnitPrice * Quantity) DESC
            ) AS ranking
    FROM sales
    WHERE CustomerID IS NOT NULL
    GROUP BY 
        CustomerID,
        DATE_PART('year', InvoiceDate),
        DATE_PART('month', InvoiceDate)
) ranked_sales
WHERE ranking = 1
ORDER BY PurchaseYear DESC, PurchaseMonth DESC;
```

Hier passiert eine Menge Action! Die wohl größte ist die Window Function `RANK` . Hierbei geht ein Fenster über einzelne Partitionen. Eine Partition ist in dem Beispiel definiert als Gesamtausgaben pro `CustonerId` pro Monat.

Ein Interessanter Fakt ist noch, dass es keiner weiteren Unterteilung zwischen Artikelpositionen und Kaufauftrag bedarf, da am Ende ohnehin alles summiert wird.

Die Rückgabe sieht dann so aus:
| customerid | purchaseyear | purchasemonth | totalpurchasevalue |
| ---------- | ------------ | ------------- | ------------------ |
| 16000      | 2011         | 12            | 12393.70           |
| 17450      | 2011         | 11            | 27837.45           |
| 18102      | 2011         | 10            | 52681.27           |
| 17450      | 2011         | 9             | 70246.50           |
| 14646      | 2011         | 8             | 39655.81           |
| 14156      | 2011         | 7             | 26464.99           |
| 18102      | 2011         | 6             | 41959.44           |
| 14646      | 2011         | 5             | 28408.14           |
| 14298      | 2011         | 4             | 9656.85            |
| 14646      | 2011         | 3             | 21462.40           |
| 14646      | 2011         | 2             | 22752.46           |
| 14646      | 2011         | 1             | 26476.68           |
| 18102      | 2010         | 12            | 27834.61           |

### Einmal durchatmen!
Natürlich wurden Ergebnisse bis hier hin weder interpretiert, noch großartig für weitere Zwecke genutzt. Aber die Fragestellungen an sich sind in dem Kontext durchaus sinnvoll und werden in vielen Situationen gestellt.

Konkret im Falle vom e-Commerce ist es wichtig seine Kundinnen und Kunden zu kennen.
Die Fragestellungen im einzelnen zielen auch ein wenig auf den Churn ab. Wann verlassen Kundinnen und Kunden die Platform? Wer verlässt sie? Wer davon ist wichtig?
Daraus kann eine Strategie entwickelt werden sie zurück zu gewinnen.

### gemessen am Vormonat: zu welchen Uhrzeiten sollte besonders viel Support-Personal verfügbar sein?
Ziel ist es in Abhängigkeit von Uhrzeit und Wochentag eine Vorhersage zu treffen, wie viel Personal am besten zu welcher Uhrzeit verfügbar sein soll.

```sql
WITH max_date AS (
    SELECT MAX(InvoiceDate) AS max_invoice_date FROM sales
)
SELECT 
    EXTRACT(DOW FROM InvoiceDate) AS DayOfWeek,
    EXTRACT(HOUR FROM InvoiceDate) AS Hour,
    COUNT(*) AS NumPurchases
FROM sales
WHERE 
    InvoiceDate BETWEEN 
        (SELECT max_invoice_date FROM max_date) - INTERVAL '2 months'  
        AND (SELECT max_invoice_date FROM max_date) - INTERVAL '1 months'
GROUP BY DayOfWeek, Hour
ORDER BY DayOfWeek, Hour
```

Auch hier wieder ein cooles Feature welches nicht extra gebaut werden muss: der Tag wird einfach aus dem Datum gezogen. Die Rückgabe sollte so aussehen:
| dayofweek | hour | numpurchases |
| --------- | ---- | ------------ |
| 0         | 9    | 3            |
| 0         | 10   | 319          |
| 0         | 11   | 1358         |
| 0         | 12   | 2224         |
| 0         | 13   | 2005         |
| 0         | 14   | 1722         |
| 0         | 15   | 1574         |
| 0         | 16   | 1013         |
| 1         | 7    | 15           |
| 1         | 8    | 227          |
| 1         | 9    | 657          |
| 1         | 10   | 944          |

Der Wochentag 0 ist dabei der Sonntag. Ausgehend von dieser Analyse könnte eine Prognose für den aktuellen Monat bzw. für die Verteilung aller Käufe pro Tag und Stunde im aktuellen Monat gemacht werden.

### Wie ist der Trend der Verkäufe pro Monat?
Ziel dieser Frage ist es konkret herauszufinden, wie hoch die Differenz des verkauften Warenwerts zum Vormonat ist. Daraus resultierend kann dann auf einen Blick bewertet werden, ob die Umsätze steigen, oder nicht.

```sql
WITH monthly_totals AS (
    SELECT
        DATE_TRUNC('month', InvoiceDate) AS PurchaseMonth,
        SUM(UnitPrice * Quantity) AS TotalPurchaseValue
    FROM sales
    GROUP BY DATE_TRUNC('month', InvoiceDate)
)
SELECT
    current_month.PurchaseMonth,
    current_month.TotalPurchaseValue AS CurrentMonthValue,
    current_month.TotalPurchaseValue - previous_month.TotalPurchaseValue AS Difference
FROM
    monthly_totals current_month
LEFT JOIN
    monthly_totals previous_month 
    ON current_month.PurchaseMonth = (previous_month.PurchaseMonth + INTERVAL '1 month')
ORDER BY current_month.PurchaseMonth DESC;
```

Diese Query ist deutlich länger, als sie sein müsste. Allerdings kann ihr vergleichsweise gut gefolgt werden. Schön finde ich die Art und weise, wie die Differenz pro Monat zum Vormonat über ein einfaches `LEFT JOIN` gebildet wird. Wichtig zu beachten ist, dass es für den ersten Monat der Tabelle keine Differenz zum Vormonat gibt und der Wert hier entsprechend `NULL` ist. 

Alternativ kann genau die gleiche Frage durch die Verwendung einer Window Function beantwortet werden:
```sql
SELECT
    PurchaseMonth,
    TotalPurchaseValue AS CurrentMonthValue,
    TotalPurchaseValue - LAG(TotalPurchaseValue, 1, TotalPurchaseValue) OVER (ORDER BY PurchaseMonth) AS Difference
FROM (
    SELECT
        DATE_TRUNC('month', InvoiceDate) AS PurchaseMonth,
        SUM(UnitPrice * Quantity) AS TotalPurchaseValue
    FROM sales
    GROUP BY DATE_TRUNC('month', InvoiceDate)
) monthly_totals
ORDER BY PurchaseMonth DESC;
```

Beide Resultate sind äquivalent:
| purchasemonth | currentmonthvalue | difference  |
| ------------- | ----------------- | ----------- |
| 2011-12-01    | 433668.01         | -1028088.24 |
| 2011-11-01    | 1461756.25        | 391051.58   |
| 2011-10-01    | 1070704.67        | 51017.048   |
| 2011-09-01    | 1019687.622       | 336965.112  |
| 2011-08-01    | 682722.51         | 1422.399    |
| 2011-07-01    | 681300.111        | -9823.009   |
| 2011-06-01    | 691123.12         | -32210.39   |
| 2011-05-01    | 723333.51         | 230126.389  |
| 2011-04-01    | 493207.121        | -190059.959 |
| 2011-03-01    | 683267.08         | 185204.43   |
| 2011-02-01    | 498062.65         | -61937.61   |
| 2011-01-01    | 560000.26         | -188956.76  |
| 2010-12-01    | 748957.02         | 0.00        |

### Wie gut muss das eigene SQL in Zeiten von ChatGPT eigentlich noch sein?
Als Entwickler in Python tendiere ich dazu Pandas und ein Jupyter Notebook als Datenbank UI zu nutzen. Dabei habe ich persönlich schon oft vergessen, wie umfangreich die Befehle der Query Languages sein können.

Im Falle von Django greife ich sogar auf ein ORM zurück und schreibe oft gar keine nativen Datenbankabfragen.

Was ich damit sagen will: auch ohne ChatGPT habe ich bislang sehr wenig SQL geschrieben. Zu meiner Verteidigung kann ich aber auch sagen, dass ich mit einer Vielzahl an Datenbanken arbeite und mich dabei nicht immer von Beginn an in die Eigenheiten der einzelnen Datenbanken einarbeiten wollte - erst recht dann, wenn meine Aufgabe die ich zu erledigen habe nicht all zu umfangreich ist. In eben diesen Fällen finde ich es als durchaus angebracht nach Möglichkeit die Daten als DataFrames zu laden und lediglich die grundlegenden SELECT und WHERE Statements zu kennen. Von da an ist Pandas oft meine Lösung für alle Datenbanken gewesen. Und diese Argumentation finde ich durchaus legitim.

ChatGPT ermöglicht mir nun meine Infos schneller und direkt aus den Datenbanken zu ziehen und mich vielleicht sogar mehr mit ihnen zu beschäftigen.
Ich glaube nicht, dass es durch ChatGPT weniger nützlich ist SQL zu lernen. Gerade als Python Entwickler kommt man lange ohne SQL aus! Daran ändert auch ChatGPT nichts.
Ich glaube aber, dass ich durch ChatGPT nun auch verstärkt SQL schreiben werde. Und dadurch "nebenbei" besser werde.

Und vermutlich bin ich damit nicht ganz alleine.

## Beispiel 2: Kapazitätsplanung einer Onlineplattform
Der folgende Datensatz ist kein berühmter Datensatz von Kaggle oder von einer anderen Plattform. Es sind echte Daten aus einem echten Projekt.

Im konkreten Fall bearbeiten Mitarbeiterinnen und Mitarbeiter digitale Anträge. Die Aufgabe wird es sein, eine Vorhersage darüber zu treffen, wie viele Anträge in den kommenden drei bis neun Tagen eingehen werden.

Das Notebook zu diesem Projekt findest du hier: **TBD**

_In diesem Beitrag versuche ich die Entwicklung des Notebooks für einen Menschen logisch zu erklären. Es kann hier und da Abweichungen vom Notebook geben._

_Der zugehörige Datensatz ist Stand heute noch nicht veröffentlicht. Dieser Beitrag und das Notebook sind also eher zum lesen und nicht zum nachmachen._

### Projektsetup
Eine Krankheit von Jupyter Notebook Projekten ist, dass ständig `imports` dort gemacht werden, wo sie gerade benötigt werden. Was das angeht, so würde ich gerne für meinen ganz persönlichen Frieden diese als erstes erledigen. Auch wenn jetzt noch nicht klar ist, was wann wofür genutzt wird:

```python
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

from statsmodels.tsa.holtwinters import ExponentialSmoothing
```

Nichts wildes. `Pandas` für die Dataframes, `Matplotlib` zum plotten, `datetime` , weil wir mit Daten arbeiten und `holtwinters` aus `statsmodels` zur Vorhersage der Daten.

Nun werden die Daten als Dataframe geladen und aufgearbeitet:
```python
path = "data.csv"
df = pd.read_csv(path, header=None, names=["id", "ts"])

df['ts'] = pd.to_datetime(df['ts'])
```

Schau' dir gerne kurz an, wie die Zeitreihen aussehen:
```
> df.head()

    id      ts
0   35831   2022-08-09 06:26:06.089817+00
1   35837   2022-08-09 06:41:35.756775+00
2   35862   2022-08-09 07:19:33.123856+00
3   35846   2022-08-09 06:49:17.813852+00
4   35854   2022-08-09 07:08:02.14196+00
```
Wie du siehst einfach nur eine numerische ID und einen Zeitstempel, dies können wir im folgenden auch Log nennen.


![enter image description here](https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/timeseries-analysis-random-introduction/basic_plot.png)


Theoretisch ist hier mal platz ein bisschen genauer rein zu schauen. Ich bin gerade allerdings etwas faul und unter Zeitdruck, also ab zum nächsten Schritt.

### Datenaufbereitung
Nun sollen die Logs gruppiert werden. Wir brauchen nicht die Information, welche ID zu welchem Zeitpunkt bearbeitet wurde. Wir wollen eher wissen in welchen Zeitintervallen welche IDs bearbeitet wurden. Daher wird das Dataframe jetzt als ein Stundenintervall gruppiert.

```python 
hourly_counts_df = (
    df
    .groupby(df['ts'].dt.to_period('H'))
    .size()
    .to_frame(name='count')
)

# Parse index column to timestamp for being able to use datetime operations
hourly_counts_df.index = hourly_counts_df.index.to_timestamp()

# Add missing timeslots and fill values with 0
fixed_freq_index = pd.date_range(
    start=hourly_counts_df.index.min(),
    end=hourly_counts_df.index.max(),
    freq='H',
)
hourly_counts_df = hourly_counts_df.reindex(
    fixed_freq_index,
    fill_value=0
)
hourly_counts_df.index.name = 'ts'
```

In diesem Abschnitt wurden die Logs zuerst in ein Stundenintervall gebracht. Anschließend wurde für spätere Operationen die Indexspalte zu einem `datetime` kompatiblen Format geparsed.
Theoretisch könnte man hier schon fertig sein. Das Problem in den Daten ist, dass die Logs nicht gleichmäßig auf jede Stunde des Tages verteilt wurde. Mit derartigen Zeitreihen wird am besten gearbeitet, wenn die Frequenz gleichmäßig ist. Im Datensatz fehlen z.B. oft Einträge zwischen 22 und 4 Uhr morgens. Diese werden mit dem Wert 0 gefüllt.

### Vorhersage mit Holt-Winters
Jetzt kann es schon an die Vorhersage gehen! Für diese Analyse wird einfach Holt-Winters genutzt. Eine Methode der Zeitreihen-Vorhersage, die entwickelt wurde, um saisonale Muster und Trends in Daten in der Vorhersage zu berücksichtigen. 

#### Trainings - Test Split
Wie im Machine Learning üblich, werden die Daten in Trainings- und Testdaten gesplittet. In vielen Methoden werden Daten einfach zufällig in einem 80/20 Verhältnis aufgeteilt. Nicht aber hier.
Es werden eigentlich nur etwa doppelt so viele Daten benötigt, wie die, die Vorhergesagt werden. Außerdem soll die Reihenfolge der Daten erhalten bleiben.

```python
def get_train_test_split(df, forecasting_day_range, forecasting_multiplicator):
    train_start_date = df.index.max() - timedelta(days = forecasting_day_range * forecasting_multiplicator)
    test_start_date = df.index.max() - timedelta(days = forecasting_day_range)

    train_data = df.loc[train_start_date:test_start_date]
    test_data = df.loc[test_start_date:]

    return train_data, test_data


FORECASTING_DAY_RANGE = 9
FORECASTING_MULTIPLICATOR = 3

train_data, test_data = get_train_test_split(
    hourly_counts_df, FORECASTING_DAY_RANGE, FORECASTING_MULTIPLICATOR
)
```

Nun können wir auch schon mit dem Forecasting anfangen und ein Model trainieren:
```python
fitted_model = ExponentialSmoothing(
    train_data['count'],
    trend='add',
    seasonal='add',
    seasonal_periods = 7 * 24
).fit()
```

Und predicten:
```python
DAYS_TO_FORECAST = 9

test_predictions = fitted_model.forecast(DAYS_TO_FORECAST*24)
```

Und zuletzt noch plotten:
```python
train_data['count'].plot(legend=True, label="TRAIN", figsize=(12, 8))
test_data['count'].plot(legend=True, label="TEST")
test_predictions.plot(legend=True, label="PREDICTION")
```

![enter image description here](https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/timeseries-analysis-random-introduction/forecast.png)
