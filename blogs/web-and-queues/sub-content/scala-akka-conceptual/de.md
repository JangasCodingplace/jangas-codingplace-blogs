# Scala & Akka
Scala ist eine sowohl funktionale- als auch objektorientierte Programmiersprache. Aus meiner persönlichen Sicht, würde ich Scala aber niemals objektorientiert nutzen.

Nach meinem Gefühl, ist die Community der Programmiersprache sehr klein. Es gibt einige große und bekannte Projekte die aus Scala aufgebaut sind, z.B. X (ehem. Twitter) oder auch Apache Spark, aber die Programmiersprache ist bei weitem nicht so verbreitet wie Python.

Scala ist eine Sprache der Java Virtual Machine (JVM), zu der z.B. auch Kotlin gehört. Und auch Kotlin ist gefühlt wesentlich weiter verbreitet als Scala.

Ich glaube, das große Problem von Scala ist, dass die Programmiersprache, richtig angewendet, wesentlich schwerer zu lernen ist, als andere Sprachen wie Python, Java oder JavaScript. Ich glaube aber auch, dass der funktionale Programmierstil wesentlich schwieriger zu verstehen ist, als der imperative & objektorientierte.

Während bei Python oft so programmiert wird, wie gedacht wird (imperativ - erst passiert das, dann das, dann das, .... ), ist der Ansatz in Scala anders. Aber da gleich ein paar Beispiele.

Ich kann abschließend sagen, dass ich Scala zwar wesentlich seltener nutze als Python und bei weitem kein Scala Guro bin - ich stehe in meiner Scala Reise noch sehr, sehr weit am Anfang - aber Scala hat meinen Python Programmierstil nachhaltig beeinflusst und meiner Meinung nach auch verbessert.

Im folgenden werden folgende Punkte vorgestellt:
- Funktionale Programmierung
- Scala Syntax - implicits
- Scala Syntax - Pattern Matching (tbd)
- Actor System & Scala Akka
- Play Framework
- Repository Pattern

Ziel ist es nicht, dir die Sprache Scala zu erklären. Mir ist wichtig, dass die verwendeten Konzepte klar werden und dass der Code grob zu verstehen ist - insbesondere für Python Entwicklerinnen und Entwickler.

## Funktionale Programmierung
Die funktionale Programmierung kann auf drei Punkte reduziert werden:
1. Alles ist eine Expression
2. Variablen sind unveränderlich
3. Eine Funktion hängt nur von ihrer Signatur ab

Typische Aktionen wie `x++` bzw `x+=1` werden in der funktionalen Programmierung streng genommen verboten. Auch Schleifen wie
```python
a = []
for x in range(10):
    a.append(x)
```
werden in der funktionalen Programmierung eher iterativ ausgedrückt. Das sind Dinge, die etwas gewöhnungsbedürftig sind. Nach einigen Jahren exklusiver Python Entwicklung fällt es mir auch extrem schwer diese anzunehmen!

Eine sehr angenehme Eigenschaft der funktionalen Programmierung ist aber, dass Seiteneffekte streng genommen verboten sind. Das spielt ein wenig in den ersten der beiden genannten Punkte "Alles ist eine Expression". Einen Seiteneffekt hat man typischerweise bei einem Statement - `a.append(x)` aus der Schleife zuvor ist so ein Statement. Indem Fall wird der Liste a ein Element hinzugefügt, ohne dass `a.append(x)` einen Rückgabewert hat. Der Wert `a` wird zusätzlich verändert - das ist ein Seiteneffekt.

Das sind Konzepte, denen man sich in der Programmierung mit Scala bewusst sein sollte.
Anbei fällt mir auf, dass ich Scala Code Konzepte durch Python Code versuche zu erklären. Das liegt vermutlich daran, dass ich primär Python Entwickler bin und Scala einfach lernen *möchte*, mich mit vielen Dingen aber noch recht schwer tue - so geht die Python Syntax schlicht leichter von der Hand. Ich werde fortlaufend immer mal wieder zwischen den Sprachen hin- und her springen. 

## Implicits
In Scala gibt es die Möglichkeiten Werte explizit sowie implizit zu übergeben. Hier einfach erstmal ein Beispiel, was ich mit explizit meine:
```scala
def getConfig: Config = ???
def doSth(config: Config): Int = ???

def execSth: Int = {
    val config: Config = getConfig
    doSth(config)
}
```

*Kurze Anmerkung für die Python Entwicklerinnen und Entwickler unter uns: In Scala ist `???` einfach gleichzusetzen mit "nicht implementiert". So kompiliert der Code, obwohl die Methode noch nicht implementiert ist. Ein bisschen so wie Pythons `...` bzw. `pass`. Auch Funktionen müssen nicht mit `()` aufgerufen- bzw. ausgeführt werden, wenn sie keine Übergabeparameter haben. Und es gibt in Scala keinen `return` - es wird einfach die letzte Expression bzw. Variable zurück gegeben*

Nun mal ein Beispiel für implizite Übergabeparameter:
```scala
def getConfig: Config = ???
def doSth(implicit config: Config): Int = ???

def execSth: Int = {
    implicit val config: Config = getConfig
    doSth
}
```
eine impliziert initialisierte Variable wird also automatisch übergeben. Ich werde das jetzt nicht bis ins Detail durch exerzieren. Für die Leute die es Challengen wollen aber drei Punkte:
- es wird nur eine Variable mit dem korrekten Typ implizit übergeben
- werden mehrere Variablen mit dem selben Typen implizit initialisiert, wird die zuletzt initialisierte Variable implizit übergeben
- sollten sowohl explizite, als auch implizite Variablen notwendig sein, können sie super voneinander getrennt werden: `def sample(expl1: Int, expl2: Int, expN: Int)(implicit impl1: Int, impl2: String)` - zwei implizite Variablen des selben Datentypen sind natürlich wenig vorteilhaft ...

Was bringt das? Mag sich die eine oder andere Person jetzt Fragen. Ich nutze das Feature oft um repetitive Variablen nicht ständig übergeben zu müssen. Dazu einfach mal ein Beispiel:

```scala
def getConfig: Config = ???

def doSth1(implicit config: Config): Int = ???
def doSth2(implicit config: Config): Int = doSth1
def doSth3(implicit config: Config): Int = doSth2

def execSth: Int = {
    implicit val config: Config = getConfig
    doSth3
}
```

Ist jetzt eventuell unvorteilhaft dargestellt, aber oft gibt es diese Boilerplate Argumente, die immer wieder in Methoden gegeben werden. In meiner Arbeit mit Spark reiche ich z.B. oft meinen SparkContext durch die Funktionen anstelle ihn mir immer mit dem selben Code Snippet wieder zu holen. In Scala könnte ich dies als implizite Variable deklarieren, würde die Variable zwar noch immer durch reichen, müsste es aber nicht immer explizit schreiben.

Der gewiefte Python Entwickler könnte nun auf die Idee kommen zu sagen: "Man kann sich ja auch einfach die Variablen importieren! Ist doch das selbe!"

Dem müsste ich entgegnen: kann man. Aber dann ist der Code nicht mehr streng-funktional. Auf der anderen Seite möchte ich aber auch nicht mit dem erhobenen Zeigefinger argumentieren. Was sich nicht natürlich anfühlt, muss auch nicht verwendet werden. Es ist aber schön zu sehen, dass Scala die funktionale Programmierung mit derartigen Sprachfeatures *angenehmer* macht.

## Actor System & Scala Akka
Das Actor System wurde in den 1970er Jahren entwickelt und bezieht sich auf das Design und die Ausführung von Nebenläufigkeit und Verteilung in Softwareanwendungen. Wesentliche Elemente des Actor System sind die Actoren, Nachrichten und Mailboxen. Ein Actor ist eine Basiseinheit zur Programmlogik. Er kann Nachrichten senden und empfangen. Wird aktuell eine Nachricht verarbeitet, wird jede weitere Nachricht in die Mailbox gelegt.

Die Actoren sind dabei einem Threadpool ausgeführt und sind keinem festen Thread zugewiesen. Sobald eine Nachricht von einem Actor verarbeitet wurde, kann dieser den Thread für einen anderen Actor frei machen.

In vielen anderen Konzepten muss entweder viel mit Callbacks gearbeitet werden, Logikeinheiten werden festen Threads zugewiesen oder Threads müssen gar für einzelne Logikeinheiten extra aufgemacht werden. Im Falle eines Actor Systems ist das wesentlich angenehmer.

Eine weitere typische Eigenschaft des Actor-Systems ist die Hierarchie. Actors können andere Actors überwachen und erschaffen. Im Falle eines Fehlers, kann dieser durch die Hierarchie nach oben eskaliert werden. So können Maßnahmen ergriffen werden, diesen Fehler zu beheben - z.B. den entsprechenden Actor neu starten oder einen alternativen Ablauf für die fehlerhafte Nachricht einschlagen.

Das Akka Framework implementiert das Actor Model in der Programmiersprache Scala, bzw. in der JVM. Für mich gab es einst zwei Gründe mich mit der Programmiersprache Scala zu beschäftigen:
1. Spark
2. Akka

Es ist nicht ganz einfach zu sagen, welche Funktionen und Features in die Definition eines Actor Systems gehören und welche exklusiv für Akka gelten. An der Stelle ist es auch nicht wichtig. Lass mich dir noch einige weitere Vorteile und Features von Actorsystemen bzw. Akka nennen.

- **Backpressure Handling** - das Actor System ist gar nicht so verschieden von zwei verschiedenen Systemen, die durch eine Queue oder einen Servicebus miteinander verbunden sind. Wenn Service A allerdings mehr Nachrichten in eine Queue schreibt, als Service B verarbeiten kann, stellt sich die Frage, was mit den Nachrichten passiert, die nicht verarbeitet werden können, nachdem sie geschickt wurden bzw. was mit Nachrichten passieren soll, die nicht mehr verarbeitet können werden, bevor sie geschickt wurden. In Akka gibt es verschiedene Strategien dieses Problem zu lösen. Der allgemeine Begriff hierfür ist entsprechend Backpressure Handling
- **Ask / Tell Pattern** - Es gibt zwei Wege, um eine Nachricht an einen Actor zu schicken. Entweder als `Tell` - hier gilt "Fire and Forget" oder als `Ask` - hier kann auf eine Rückgabe gewartet werden. Dabei muss beachtet werden, dass Actoren zwar grundsätzlich nebenläufig sind, allerdings bearbeiten sie nur eine Nachricht zur selben Zeit, wodurch s.g. Race Conditions vermieden werden können. Für das Ask-Pattern bedeutet dies allerdings, dass ein Actor mit der Verarbeitung der nächsten Nachricht "lange" warten muss. Es wird daher oft mit einem Tell - Tell Pattern gearbeitet. Actor A schickt Actor B mit dem Tell pattern eine Nachricht. Actor B verarbeitet die Nachricht und schickt an den Absender der Nachricht (in dem Fall Actor A) wieder eine Nachricht. Diese Nachricht kann in einer womöglich anderen Routine verarbeitet werden, als die Ausgangsnachricht.

Akka bietet viele interessante Ansätze. Besonders angenehm finde ich, dass sich die nebenläufige Programmierung in Akka sehr "natürlich" anfühlt und dass es die "reaktive" Programmierung deutlich vereinfacht. Zudem gibt es mit Akka, Akka-HTTP, Akka-Persist, Akka-Streams und Distributed Akka viele verschiedene Erweiterungen um z.B. Internetseiten oder APIs zu bauen, Mailboxen zu persistieren oder das ganze Actor System über verschiedene physische Maschinen hinweg zu verteilen.

## Play
Das Play Framework ist ein Scala geschriebenes Webframework welches an verschiedenen Stellen auf Akka-HTTP und Akka-Streams zurück greift.	

Play ist recht minimal und schreibt wenig vor. Durch die Integration von Akka, sind Anwendungen von Play natürlicherweise nebenläufig und Reaktiv. Darüber hinaus bleibt es dir überlassen, in welchem Pattern du programmieren möchtest: Model View Controller (MVC), Repoisitory Pattern, oder etwas ganz anderes. Im konkreten Projekt wird das Repository Pattern genutzt.

## Repository Pattern
Das Repository-Pattern ist in der Softwareentwicklung ein etabliertes Designmuster, das den Datenzugriff von der Geschäftslogik einer Anwendung trennt. Dieser Ansatz bietet nicht nur eine konsistente Methode, um auf Daten zuzugreifen, sondern ermöglicht auch eine höhere Flexibilität. Wenn beispielsweise die Datenquelle einer Anwendung geändert werden muss, reicht es aus, nur das entsprechende Repository zu aktualisieren, ohne tiefgreifende Änderungen in der gesamten Anwendung vornehmen zu müssen.

Das Hinzufügen eines Repository-Patterns zu sehr einfachen Projekten kann zu unnötigem Overhead führen. Das wird auch im Dev-Guro Beispielprojekt deutlich. Meiner Meinung nach gibt es im Repository Pattern aber eine klare Struktur. Insbesondere bei Frameworks wie dem Play Framework, oder auch den Python Frameworks Flask oder FastAPI kann es von Vorteil sein, eine "klare" Struktur durch ein "einfaches" Pattern vorgegeben zu kriegen. Im Repository Pattern gibt es folgende Kernteile:
- **Model oder Entity** Das Model (oft als Entität bezeichnet) repräsentiert die Datenstruktur oder das Geschäftsobjekt, mit dem das System arbeitet. Ein Beispiel könnte ein `User`-Model sein, das Eigenschaften wie `id`, `name` und `e_mail` hat.
- **Repository** Das Repository stellt eine Abstraktion des Datenzugriffs für ein spezifisches Model bereit. Im Falle von Scala ist das Repository oft ein Interface `trait`. Es definiert eine Schnittstelle (oft in stark typisierten Sprachen) mit Methoden, die den Zugriff auf die Daten ermöglichen, wie z.B. `findById`, `save`, `delete` usw. ohne diese Methoden konkret im Repository implementiert werden.
- **Repository Implementierung** -während das Repository die abstrakte Schnittstelle definiert, stellt die Repository Implementierung den tatsächlichen Code bereit, der diese Methoden ausführt. Diese Implementierung kann spezifisch für eine bestimmte Datenquelle sein, z.B. eine SQL-Datenbank, ein NoSQL-System oder sogar ein Remote-API-Endpunkt und ist leicht auszutauschen.
- **Controller**: Der Controller interagiert in der Regel direkt mit dem Repository und nimmt Anfragen vom Client entgegen, ruft die notwendigen Daten über das Repository ab (oder aktualisiert sie) und gibt dann eine Antwort oder eine Ansicht and den Client zurück.
