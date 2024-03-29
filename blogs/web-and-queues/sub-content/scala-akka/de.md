# Scala Play & Akka
[//]: # (START CUSTOM SCRIPT)
[//]: # (START MARKDOWNREF)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/assets/project-description/de.md)
[//]: # (END MARKDOWNREF)
[//]: # (END CUSTOM SCRIPT)




## Überblick
In dieser Lösung wird Scala mit dem Play Webframework verwendet. Da es keine vollständige Liste an 
Technologien geben kann, wird ein LLM zur Informationsextraktion der Logs verwendet, im konkreten 
GPT-3.5-Turbo von OpenAI.

Allerdings dauert die Abfrage gegen diese API verhältnismäßig lange und unter Last schlägt sie auch
gerne mal fehl. Daher soll die Nutzerinteraktion (speichere Log) von der Informationsextraktion
durch OpenAI getrennt werden.

Diese Trennung erfolgt über Scala Akka. Akka ist ein Scala-Framework und folgt dem Actor Model, welches 
im folgenden noch erklärt wird. Kurz gefasst handelt es sich hier um ein nebenläufiges Programm welches
Über Nachrichten über verschiedene Threads hinweg kommuniziert.

Hier liegt also ein rein monolithischer Ansatz vor, der kein weiteres Tooling bedarf.



### Die Vorteile
- natürliche Nebenläufigkeit, ohne dass sie extra programmiert werden muss / ohne dass sie sich unnatürlich anfühlt
- durch distributed Akka ist theoretisch eine vertikale Skalierung (eine Maschine fetter machen) und horizontale Skalierung (Anwendung auf mehrere Maschine verteilen) möglich, ohne die Codebasis zu trennen
- Im vergleich zu anderen nebenläufigen Ansätzen muss kein Thread parallel zur Anwendung am leben gehalten werden. Das Actor Model geht auf einen Threadpool zurück. Die Actoren spawnen einfach auf einem freien Thread, führen ihre Aktion durch und verschwinden wieder. Es ist nicht schlimm für den Actor, wenn eine Aktion terminiert. Das ist eine Sache, die ich in den serverless Functions sehr schätze. Damit ist das Programm insgesamt etwas ausfallsicherer



### Die Nachteile
- gerade im Vergleich zu Django, aber auch zu vielen anderen Frameworks bietet insbesondere Play wenig built-in Funktionen und ist deutlich schwerer mit Bibliotheken zu erweitern als insbesondere Python Bibliotheken wie Flask oder FastAPI
- Scala als Programmiersprache ist ehrlich gesagt ein Nachteil. Es gibt einfach vergleichsweise wenig Leute, die in Scala programmieren
- steile Lernkurve (ich behaupte, dass insbesondere die Verwendung von `implicits` oder auch der `anorm` Bibliothek sowie viele weitere Sprachfeatures absolut nicht selbsterklärend sind.



### Nachtrag zur Skalierbarkeit
Ein Nachtrag zur Skalierbarkeit: für mich ist die Skalierbarkeit selten ein relevantes Argument. 
Zum einen muss nicht jedes Projekt skalierbar sein, zum anderen liegt meiner Erfahrung nach auch 
viel im Code und nicht zwingend an der Programmiersprache oder im Tooling. Wir müssen nicht darüber 
streiten, dass Scala als Programmiersprache in der Ausführung in vielen Fällen deutlich schneller sein 
wird, als es Python ist. Wir können aber drüber streiten, ob es für den Endnutzer so spürbar ist und ob, 
falls es überhaupt notwendig wird, es nicht einfach effizienter ist mehr Geld für die Infrastruktur 
auszugeben, als einen Ansatz wie diesen zu wählen bei dem man eventuell günstigere Infrastruktur Kosten 
hat, aber einen deutlich weniger verbreiteten Ansatz hat.

Generell bin ich der festen Überzeugung, dass es bei Projekten die wirklich skalierbar sein müssen, 
mehr als nur einen klugen Kopf gibt, der darüber nachdenkt. Auf dieser ebene, bei diesem Projekt müssen wir 
uns über diesen Punkt nicht ernsthaft unterhalten. Skalierbarkeit ist einfach kompliziert und oft 
missverstanden.



## Theoretisches und Konzepte
Scala ist eine sowohl funktionale- als auch objektorientierte Programmiersprache. Aus meiner persönlichen 
Sicht, würde ich Scala aber niemals objektorientiert nutzen.

Nach meinem Gefühl, ist die Community der Programmiersprache sehr klein. Es gibt einige große und bekannte 
Projekte die aus Scala aufgebaut sind, z.B. X (ehem. Twitter) oder auch Apache Spark, aber die Programmiersprache 
ist bei weitem nicht so verbreitet wie Python.

Scala ist eine Sprache der Java Virtual Machine (JVM), zu der z.B. auch Kotlin gehört. Und auch Kotlin ist 
gefühlt wesentlich weiter verbreitet als Scala.

Ich glaube, das große Problem von Scala ist, dass die Programmiersprache, richtig angewendet, wesentlich schwerer 
zu lernen ist, als andere Sprachen wie Python, Java oder JavaScript. Ich glaube aber auch, dass der funktionale 
Programmierstil wesentlich schwieriger zu verstehen ist, als der imperative & objektorientierte.

Während bei Python oft so programmiert wird, wie gedacht wird (imperativ - erst passiert das, dann das, dann das, .... ), 
ist der Ansatz in Scala anders. Aber da gleich ein paar Beispiele.

Ich kann abschließend sagen, dass ich Scala zwar wesentlich seltener nutze als Python und bei weitem kein Scala Guro 
bin - ich stehe in meiner Scala Reise noch sehr, sehr weit am Anfang - aber Scala hat meinen Python Programmierstil 
nachhaltig beeinflusst und meiner Meinung nach auch verbessert.

Im folgenden werden folgende Punkte vorgestellt:
- Funktionale Programmierung
- Scala Syntax - implicits
- Scala Syntax - Pattern Matching (tbd)
- Actor System & Scala Akka
- Play Framework
- Repository Pattern

Ziel ist es nicht, dir die Sprache Scala zu erklären. Mir ist wichtig, dass die verwendeten Konzepte klar werden 
und dass der Code grob zu verstehen ist - insbesondere für Python Entwicklerinnen und Entwickler.



### Funktionale Programmierung
Die funktionale Programmierung kann auf drei Punkte reduziert werden:


1. Alles ist eine Expression
2. Variablen sind unveränderlich
3. Eine Funktion hängt nur von ihrer Signatur ab


Typische Aktionen wie `x++` bzw `x+=1` werden in der funktionalen Programmierung streng genommen verboten. 
Auch Schleifen wie
```python
a = []
for x in range(10):
    a.append(x)
```
werden in der funktionalen Programmierung eher iterativ ausgedrückt. Das sind Dinge, die etwas gewöhnungsbedürftig 
sind. Nach einigen Jahren exklusiver Python Entwicklung fällt es mir auch extrem schwer diese anzunehmen!

Eine sehr angenehme Eigenschaft der funktionalen Programmierung ist aber, dass Seiteneffekte streng genommen 
verboten sind. Das spielt ein wenig in den ersten der beiden genannten Punkte "Alles ist eine Expression". Einen 
Seiteneffekt hat man typischerweise bei einem Statement - `a.append(x)` aus der Schleife zuvor ist so ein Statement. 
Indem Fall wird der Liste a ein Element hinzugefügt, ohne dass `a.append(x)` einen Rückgabewert hat. Der Wert `a` wird 
zusätzlich verändert - das ist ein Seiteneffekt.

Das sind Konzepte, denen man sich in der Programmierung mit Scala bewusst sein sollte.
Anbei fällt mir auf, dass ich Scala Code Konzepte durch Python Code versuche zu erklären. Das liegt vermutlich daran, 
dass ich primär Python Entwickler bin und Scala einfach lernen *möchte*, mich mit vielen Dingen aber noch recht schwer 
tue - so geht die Python Syntax schlicht leichter von der Hand. Ich werde fortlaufend immer mal wieder zwischen den Sprachen 
hin- und her springen. 



### Implicits
In Scala gibt es die Möglichkeiten Werte explizit sowie implizit zu übergeben. Hier einfach erstmal ein Beispiel, was 
ich mit explizit meine:

```scala
def getConfig: Config = ???
def doSth(config: Config): Int = ???

def execSth: Int = {
    val config: Config = getConfig
    doSth(config)
}
```

*Kurze Anmerkung für die Python Entwicklerinnen und Entwickler unter uns: In Scala ist `???` einfach gleichzusetzen mit 
"nicht implementiert". So kompiliert der Code, obwohl die Methode noch nicht implementiert ist. Ein bisschen so wie Pythons 
`...` bzw. `pass`. Auch Funktionen müssen nicht mit `()` aufgerufen- bzw. ausgeführt werden, wenn sie keine Übergabeparameter 
haben. Und es gibt in Scala keinen `return` - es wird einfach die letzte Expression bzw. Variable zurück gegeben*

Nun mal ein Beispiel für implizite Übergabeparameter:

```scala
def getConfig: Config = ???
def doSth(implicit config: Config): Int = ???

def execSth: Int = {
    implicit val config: Config = getConfig
    doSth
}
```
eine impliziert initialisierte Variable wird also automatisch übergeben. Ich werde das jetzt nicht bis ins Detail durch exerzieren.
Für die Leute die es Challengen wollen aber drei Punkte:


- es wird nur eine Variable mit dem korrekten Typ implizit übergeben
- werden mehrere Variablen mit dem selben Typen implizit initialisiert, wird die zuletzt initialisierte Variable implizit übergeben
- sollten sowohl explizite, als auch implizite Variablen notwendig sein, können sie super voneinander getrennt werden: `def sample(expl1: Int, expl2: Int, expN: Int)(implicit impl1: Int, impl2: String)` - zwei implizite Variablen des selben Datentypen sind natürlich wenig vorteilhaft ...


Was bringt das? Mag sich die eine oder andere Person jetzt Fragen. Ich nutze das Feature oft um repetitive Variablen 
nicht ständig übergeben zu müssen. Dazu einfach mal ein Beispiel:

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

Ist jetzt eventuell unvorteilhaft dargestellt, aber oft gibt es diese Boilerplate Argumente, die immer wieder in Methoden 
gegeben werden. In meiner Arbeit mit Spark reiche ich z.B. oft meinen SparkContext durch die Funktionen anstelle ihn mir 
immer mit dem selben Code Snippet wieder zu holen. In Scala könnte ich dies als implizite Variable deklarieren, würde die 
Variable zwar noch immer durch reichen, müsste es aber nicht immer explizit schreiben.

Der gewiefte Python Entwickler könnte nun auf die Idee kommen zu sagen: "Man kann sich ja auch einfach die Variablen 
importieren! Ist doch das selbe!"

Dem müsste ich entgegnen: kann man. Aber dann ist der Code nicht mehr streng-funktional. Auf der anderen Seite möchte ich 
aber auch nicht mit dem erhobenen Zeigefinger argumentieren. Was sich nicht natürlich anfühlt, muss auch nicht verwendet 
werden. Es ist aber schön zu sehen, dass Scala die funktionale Programmierung mit derartigen Sprachfeatures *angenehmer* macht.



### Actor System & Scala Akka
Das Actor System wurde in den 1970er Jahren entwickelt und bezieht sich auf das Design und die Ausführung von Nebenläufigkeit 
und Verteilung in Softwareanwendungen. Wesentliche Elemente des Actor System sind die Actoren, Nachrichten und Mailboxen. Ein 
Actor ist eine Basiseinheit zur Programmlogik. Er kann Nachrichten senden und empfangen. Wird aktuell eine Nachricht verarbeitet, 
wird jede weitere Nachricht in die Mailbox gelegt.

Die Actoren sind dabei einem Threadpool ausgeführt und sind keinem festen Thread zugewiesen. Sobald eine Nachricht von einem 
Actor verarbeitet wurde, kann dieser den Thread für einen anderen Actor frei machen.

In vielen anderen Konzepten muss entweder viel mit Callbacks gearbeitet werden, Logikeinheiten werden festen Threads zugewiesen 
oder Threads müssen gar für einzelne Logikeinheiten extra aufgemacht werden. Im Falle eines Actor Systems ist das wesentlich 
angenehmer.

Eine weitere typische Eigenschaft des Actor-Systems ist die Hierarchie. Actors können andere Actors überwachen und erschaffen. 
Im Falle eines Fehlers, kann dieser durch die Hierarchie nach oben eskaliert werden. So können Maßnahmen ergriffen werden, 
diesen Fehler zu beheben - z.B. den entsprechenden Actor neu starten oder einen alternativen Ablauf für die fehlerhafte Nachricht 
einschlagen.

Das Akka Framework implementiert das Actor Model in der Programmiersprache Scala, bzw. in der JVM. Für mich gab es einst zwei 
Gründe mich mit der Programmiersprache Scala zu beschäftigen:


1. Spark
2. Akka


Es ist nicht ganz einfach zu sagen, welche Funktionen und Features in die Definition eines Actor Systems gehören und welche 
exklusiv für Akka gelten. An der Stelle ist es auch nicht wichtig. Lass mich dir noch einige weitere Vorteile und Features 
von Actorsystemen bzw. Akka nennen.

- **Backpressure Handling** - das Actor System ist gar nicht so verschieden von zwei verschiedenen Systemen, die durch eine Queue oder einen Servicebus miteinander verbunden sind. Wenn Service A allerdings mehr Nachrichten in eine Queue schreibt, als Service B verarbeiten kann, stellt sich die Frage, was mit den Nachrichten passiert, die nicht verarbeitet werden können, nachdem sie geschickt wurden bzw. was mit Nachrichten passieren soll, die nicht mehr verarbeitet können werden, bevor sie geschickt wurden. In Akka gibt es verschiedene Strategien dieses Problem zu lösen. Der allgemeine Begriff hierfür ist entsprechend Backpressure Handling
- **Ask / Tell Pattern** - Es gibt zwei Wege, um eine Nachricht an einen Actor zu schicken. Entweder als `Tell` - hier gilt "Fire and Forget" oder als `Ask` - hier kann auf eine Rückgabe gewartet werden. Dabei muss beachtet werden, dass Actoren zwar grundsätzlich nebenläufig sind, allerdings bearbeiten sie nur eine Nachricht zur selben Zeit, wodurch s.g. Race Conditions vermieden werden können. Für das Ask-Pattern bedeutet dies allerdings, dass ein Actor mit der Verarbeitung der nächsten Nachricht "lange" warten muss. Es wird daher oft mit einem Tell - Tell Pattern gearbeitet. Actor A schickt Actor B mit dem Tell pattern eine Nachricht. Actor B verarbeitet die Nachricht und schickt an den Absender der Nachricht (in dem Fall Actor A) wieder eine Nachricht. Diese Nachricht kann in einer womöglich anderen Routine verarbeitet werden, als die Ausgangsnachricht.

Akka bietet viele interessante Ansätze. Besonders angenehm finde ich, dass sich die nebenläufige Programmierung in Akka sehr 
"natürlich" anfühlt und dass es die "reaktive" Programmierung deutlich vereinfacht. Zudem gibt es mit Akka, Akka-HTTP, 
Akka-Persist, Akka-Streams und Distributed Akka viele verschiedene Erweiterungen um z.B. Internetseiten oder APIs zu bauen, 
Mailboxen zu persistieren oder das ganze Actor System über verschiedene physische Maschinen hinweg zu verteilen.



### Play
Das Play Framework ist ein Scala geschriebenes Webframework welches an verschiedenen Stellen auf Akka-HTTP und Akka-Streams 
zurück greift.	

Play ist recht minimal und schreibt wenig vor. Durch die Integration von Akka, sind Anwendungen von Play natürlicherweise 
nebenläufig und Reaktiv. Darüber hinaus bleibt es dir überlassen, in welchem Pattern du programmieren möchtest: Model View 
Controller (MVC), Repoisitory Pattern, oder etwas ganz anderes. Im konkreten Projekt wird das Repository Pattern genutzt.



### Repository Pattern
Das Repository-Pattern ist in der Softwareentwicklung ein etabliertes Designmuster, das den Datenzugriff von der Geschäftslogik 
einer Anwendung trennt. Dieser Ansatz bietet nicht nur eine konsistente Methode, um auf Daten zuzugreifen, sondern ermöglicht 
auch eine höhere Flexibilität. Wenn beispielsweise die Datenquelle einer Anwendung geändert werden muss, reicht es aus, nur das 
entsprechende Repository zu aktualisieren, ohne tiefgreifende Änderungen in der gesamten Anwendung vornehmen zu müssen.

Das Hinzufügen eines Repository-Patterns zu sehr einfachen Projekten kann zu unnötigem Overhead führen. Das wird auch im Dev-Guro 
Beispielprojekt deutlich. Meiner Meinung nach gibt es im Repository Pattern aber eine klare Struktur. Insbesondere bei Frameworks 
wie dem Play Framework, oder auch den Python Frameworks Flask oder FastAPI kann es von Vorteil sein, eine "klare" Struktur durch 
ein "einfaches" Pattern vorgegeben zu kriegen. Im Repository Pattern gibt es folgende Kernteile:


- **Model oder Entity** Das Model (oft als Entität bezeichnet) repräsentiert die Datenstruktur oder das Geschäftsobjekt, mit dem das System arbeitet. Ein Beispiel könnte ein `User`-Model sein, das Eigenschaften wie `id`, `name` und `e_mail` hat.
- **Repository** Das Repository stellt eine Abstraktion des Datenzugriffs für ein spezifisches Model bereit. Im Falle von Scala ist das Repository oft ein Interface `trait`. Es definiert eine Schnittstelle (oft in stark typisierten Sprachen) mit Methoden, die den Zugriff auf die Daten ermöglichen, wie z.B. `findById`, `save`, `delete` usw. ohne diese Methoden konkret im Repository implementiert werden.
- **Repository Implementierung** -während das Repository die abstrakte Schnittstelle definiert, stellt die Repository Implementierung den tatsächlichen Code bereit, der diese Methoden ausführt. Diese Implementierung kann spezifisch für eine bestimmte Datenquelle sein, z.B. eine SQL-Datenbank, ein NoSQL-System oder sogar ein Remote-API-Endpunkt und ist leicht auszutauschen.
- **Controller**: Der Controller interagiert in der Regel direkt mit dem Repository und nimmt Anfragen vom Client entgegen, ruft die notwendigen Daten über das Repository ab (oder aktualisiert sie) und gibt dann eine Antwort oder eine Ansicht and den Client zurück.



## Die Implementierung



### Datenbank & Models
Als erstes kommt die Datenbank. Innerhalb von Play könnte sehr gut mit einem ORM und im MVC gearbeitet werden. Ich kenne mich dort aber zu wenig aus. Also gehe ich zurück auf echtes SQL und lege als erstes eine geeignete Datenbankstruktur an.

```SQL

CREATE TABLE IF NOT EXISTS logs (
    id SERIAL PRIMARY KEY,
    entry TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS tags (
    id SERIAL PRIMARY KEY,
    log INTEGER REFERENCES logs(id),
    type_of VARCHAR(255),
    value VARCHAR(255)
);
```

Analog dazu können auch direkt die Models in Scala angelegt werden.

```scala
case class Log(id: Long, entry: String, timestamp: Long)
case class Tag(id: Long, log: Long, typeOf: String, value: String)
```

Hier ist noch sinnvoll zu erwähnen: im echten Projekt sind die Models nicht in der selben File. Ich habe irgendwo mal gelesen, dass in der Java Welt oft nur eine Klasse pro Datei geschrieben wird. Ich weiß nicht, ob da was dran ist. Aber ich bevorzuge allgemein kleine Files und habe kein Problem damit diesem Motto einfach streng nachzugehen.


### Repository
Im aktuellen Ansatz wird ein Repository Pattern gewählt. Daher im folgenden die Interfaces für die Repositories. Auch hier wieder: im echten Projekt sind sie in getrennten Files. Für diesen Text halte ich gerne alles dicht zusammen.
```scala
trait LogRepository {
  def create(entry: String): Try[Log]
}

trait TagRepository {
  def bulkCreate(log: Long, tags: Array[String], typeOf: String): Try[Array[Tag]]
}
```

Als primärer Python Entwickler war - und ist es auch noch immer - es extrem schwer zu akzeptieren, was ich da sehe. Zwei Klassen die einfach nichts machen. Hier liegen einfach zwei Interfaces vor, die ein Objekt grundsätzlich definieren. Werden diese Interfaces vererbt, so müssen die Klassen, die diese Interfaces vererbt bekommen alle Methoden und Attribute inklusive Signaturen implementieren, sonst kompiliert das Programm nicht.

Ein ähnliches Konzept in Python gibt es durch die Abstrakten Klassen. Wobei diese auch keine Interfaces sind ...

Bezüglich der konkreten Implementierung ist die Rückgabe des Typen `Try` vermutlich recht interessant. Es gibt ein Sprichwort "Functional Components are always telling the truth." Mit anderen Worten: Es kann das erwartete Objekt zurück kommen, diese Methode könnte aber auch eine Exception werfen. Allerdings wird sich in dieser Methode nicht über das Errorhandling Gedanken gemacht. Das passiert vielleicht an dem Punkt, von dem sie aufgerufen wird.

### Log-Implementierung
Fangen wir mit der Implementierung des Log-Repositories an. Diese Methode muss eigentlich nur eine Sache können: einen Log in die Datenbank schreiben und den neuen Log zurück geben

```scala
class LogImp @Inject()(db: Database) extends LogRepository {
  def create(entry: String): Try[Log] = Try {
    db.withConnection { implicit conn =>
      SQL("INSERT INTO logs (entry) VALUES ({entry}) RETURNING id,EXTRACT(EPOCH FROM timestamp) * 1000 as unix_ts;")
        .on("entry" -> entry)
        .as((SqlParser.long("id") ~ SqlParser.long("unix_ts")).singleOpt)
    } match {
      case Some(value) => Log(value._1, entry, value._2)
      case None => throw new Exception()
    }
  }
}
```

- die Klasse "injeziert" eine Datenbank Instanz. Das wird glücklicherweise weitgehend von Play für uns übernommen - es gibt keinen Punkt, an dem die Datenbank konkret instantiiert wird. Die Konfiguration wird sich aus der Konfigurationsdatei geholt und der Rest passiert im Hintergrund.
- die Klasse `LogImp` bekommt das `LogRepository` vererbt. Das `LogRepository` ist jedoch nur ein Interface. Was bedeutet, dass alle Methoden dieses Interfaces (in dem Fall nur die `create` Methode) implementiert werden muss
- nun wird die `create` Methode implementiert. Beim programmieren hatte ich dort im ersten Moment nur `def create(entry: String): Try[Log] = ???` stehen. Das ist einfach ein Platzhalter, das Programm kann mit diesem Platzhalter noch immer gebaut werden und ich konnte mich um andere Dinge zuerst kümmern
- der Return Type `Try[Log]` beschreibt, dass im Erfolgsfall ein Log Objekt zurück gegeben wird. Es kann allerdings auch sein, dass das Programm vorher terminiert. Außerhalb dieser Methode muss sich dann Gedanken darüber gemacht werden, wie damit umzugehen ist
- `db.withConnection {implicit conn => ??? }` holt sich eine Verbindung zur Datenbank. `implicit conn` bedeutet, dass die Variable einer Methode nicht explizit übergeben werden muss, wenn sie dort auch als implizit deklariert ist
- `SQL` kommt aus der Library `anorm` und kümmert sich darum, dass Queries und Statements auf der Datenbank ausgeführt werden können
- Das SQL Statement selbst schreibt nur einen Log in die Datenbank. Da sowohl `id`, als auch `timestamp` von der Datenbank selbst gesetzt werden, kümmere ich mich darum im Scala Code nicht. Wobei das beim Timestamp durchaus kritisch sein kann. Hier muss unbedingt darauf geachtet werden, dass die Timestamps, die von der Datenbank gesetzt werden unbedingt in "der richtigen" Zeitzone vom Scala Code "verstanden" werden. Das gilt insbesondere auch für Timestamps, die ggf. vom Scala Code an anderer Stelle selbst gesetzt und in die Datenbank geschrieben werden ... mit dieser Art von Problemen habe ich Tage an Debugging verbracht ... das ist nicht schön. Da die Datenbank bestimmte Werte setzt, werden diese vom `INSERT` Statement direkt mit zurück gegeben um damit dann im Scala Code ein richtiges Objekt zu bauen
- `on` hier werden einfach nur Variablen auf das Statement gemapped und dem Statement übergeben
- `as` hier wird überlegt, wie mit der Rückgabe der Datenbank umgegangen wird. Im konkreten Fall gibt die Datenbank eine `id` des Typen `long` sowie `unix_ts` des Typen `long` zurück. `singleOpt` meint in dem Fall nur, dass maximal ein Objekt zurück gegeben wird, es kann aber auch sein, dass es nichts zurück gibt
- Das ganze Statement wird anschließend einem Pattern-Matching übergeben. Wenn das SQL Statement etwas zurück gibt, dann wird ein `Log` Objekt instantiiert und von der Methode zurück gegeben. Wenn das SQL Statement dieser Erwartung nicht entspricht, wird eine nicht näher typisierte Exception geworfen.

Ui. Ganz schön viel! Meiner Meinung nach, ist diese Methode auch nicht ganz sauber implementiert. Das SQL Statement sowie das Pattern Matching sollten vermutlich in getrennten Methoden stattfinden. Vielleicht ist das aber auch nur Geschmacksache.

### Serializer
Ein Serializer kümmert sich darum, dass ein request vom Client vernünftig zu einem Objekt geparsed wird (die `Read` Richtung) und anschließend wieder vernünftig in ein JSON Format gebracht wird (die `Write` Richtung).

```scala
object LogCreateSerializer {
  case class LogWriteInput(entry: String)
 
  implicit val logWritable: Writes[Log] = new Writes[Log] {  
    def writes(log: Log) = Json.obj(  
      "id" -> log.id,  
      "entry" -> log.entry,  
      "timestamp" -> log.timestamp  
    )  
  }  
  implicit val logReadable: Reads[LogWriteInput] = Json.reads[LogWriteInput]  
}
```
Diese Serializer wird anschließend im Controller verwendet.

### Controller
Im Controller wird die Kommunikation mit dem Client geregelt. Hierbei werden Requests in Objekte geparsed (serializiert), einem Repository übergeben und die Antwort des Repositories wieder in JSON Formate gebracht (deserializiert) und an den Client zurück gegeben.

```scala
@Singleton
class LogController @Inject()(cc: ControllerComponents, repository: LogRepository)
  extends GenericCRUDController(cc) {
  import serializers.LogCreateSerializer._

  def post = create[LogWriteInput, Log]{request =>
    repository.create(request.body.entry)
  }
}
```
- der Klasse `LogController` werden zwei Variablen injiziert: `ControllerComponents` und das zuvor definierte `LogRepository`
- die Klasse bekommt einen `GenericCRUDController` vererbt. Das ist eine von mir geschriebene Klasse, die ein wenig den Boilerplate-Code verringern soll. Letztlich kommt von ihm aber die Methode `create` auf die der `LogController` zurückgreift
- innerhalb der Methode wird das `LogCreateSerializer` Modul importiert, welche zwei implizite Variablen verfügbar macht - einen Reader und ein Writer. Diese werden für die Serializierung, sowie Deserializierung benötigt
- zuletzt wird die `create` Methode aus dem Repository aufgerufen. Dabei wird auf die implementierte Methode zurück gegangen. Auch das ist für mich als primären Python Entwickler echt hart zu begreifen... in einem Konfigurationsobjekt kapsle ich die Definierung aber ans Repository: `bind(classOf[LogRepository]).to(classOf[LogImp])` und eben das wird dem `LogController` dann injiziert.
- Wichtig: da der Expression `repository.create` keine Variable zugewiesen wird, wird diese automatisch aus der Funktion zurück gegeben

Hier noch mein Hilfscontroller `GenericCRUDController`:
```scala
trait CRUDController {
  def create[A, B](f: Request[A] => Try[B])(implicit reads: Reads[A], writes: Writes[B]): Action[A]
}

class GenericCRUDController(cc: ControllerComponents)
  extends AbstractController(cc)
    with CRUDController {

  def create[A, B](f: Request[A] => Try[B])(implicit reads: Reads[A], writes: Writes[B]) =
    Action(parse.json[A]) { request =>
      f(request) match {
        case Success(value) => Ok(Json.toJson(value))
        case Failure(exc) => BadRequest(Json.obj("message" -> exc.getMessage))
      }
    }
}
```

Über den GenericCRUDController wird die Rückgabe und das Errorhandling an den Client geregelt. Kann man so drüber streiten, ob das okay ist, oder nicht. Ich habe mich ein wenig von den Generic API Views des Django Restframeworks inspirieren lassen.

Eine Sache die ich durch Scala übrigens lernen musste ist es mit Typisierung und insbesondere generischen Datentypen umzugehen. Generische Datentypen sind in dem Fall `A` und `B`.

Viel weiter möchte ich auf meine Hilfsklasse nicht eingehen. Ich bin froh sie geschrieben zu haben. Sie ist aber nicht wirklich sauber. Ich wollte nur ein wenig die "Magie" des Codes reduzieren (oder den "Wahnsinn" weiter schüren ;-) )

## Durchatmen und testen
An dieser Stelle lohnt sich eine kurze Pause. Der Endpunkt kann nun getestet werden. Es sollten dabei Einträge ihren Weg in die Datenbank finden und an den Client zurück gegeben werden. Etwas was noch fehlt ist, dass die Logbeiträge OpenAI zur Informationsextraktion übergeben werden. Hierzu wird Akka genutzt.

Zur Erinnerung: Akka folgt dem Actor Model. Ein Actor ist eine Logikeinheit und bekommt Nachrichten zugeschickt, die verarbeitet werden sollen.


### Actor - Actions
Um Informationen aus einem Log zu extrahieren und in die Datenbank zu schreiben, werden verschiedene Etappen durchlaufen:
1. Log wird an OpenAI zur Informationsextraktion gegeben
2. Die Rückgabe von OpenAI wird zu einer Liste geparsed
3. Die Liste wird in die Datenbank geschrieben

Diese Aktionen werden auf drei Actors aufgeteilt:
- LLM Actor
- OpenAI Actor
- TagRepository Actor

Eine Aktion symbolisiert entweder die Aktion die als nächstes ausgeführt werden soll, oder die, die zuletzt ausgeführt wurde. Im folgenden einmal alle Aktionen:

```scala
object LLMActorActions {
  case class ExtractTechStack(log: Long, text: String)
  case class ExtractTechStackResponse(log: Long, llm_response: String)
}

object OpenAIPromptActorActions {
  case class ExecPrompt(identifier: Long, userInput: String, systemInput: String)
}

object TagRepositoryActorActions {
  case class BulkCreate(log: Long, tags: Array[String])
}
```

Eine Nachricht die an einen Actor geschickt wird, wird eines dieser Objekte sein. Durch Pattern-Matching weiß der Empfänger anschließend, was zu tun ist.

### OpenAI Prompt Actor
Der Open AI Prompt Actor hat die Aufgabe den Log und eine Systemanweisung an OpenAI zu schicken.

```scala
class OpenAIPromptActor(implicit config: Configuration) extends Actor {
  import akkaSystem.actorActions.OpenAIPromptActorActions._

  implicit val system: ActorSystem = context.system

  override def receive: Receive = {
    case ExecPrompt(identifier, userInput, systemInput) =>
      sender() ! ExtractTechStackResponse(identifier, OpenAIPrompt(userInput, systemInput).getResponseText)
    case _ => println("Not supported Action")
  }
}
```
- der `OpenAIPromptActor` bekommt implizit die Konfiguration der Anwendung übergeben. Darin liegt insbesondere der Access Token für open AI der für die Informationsextraktion benötigt wird
- der `OpenAIPromptActor` bekommt den `Actor` vererbt - eine Instanz die direkt von Akka kommt
- `import akkaSystem.actorActions.OpenAIPromptActorActions._` sind die definierten Aktionen bzw. Typen die Nachrichten haben können, die anschließend zu einer Aktion führen
- anschließend wird das `system` implizit initialisiert. Das ist dabei ein `ActorSystem` und wird von meinem Hilfsmodul benötigt
- die `receive` Methode ist das Herzstück eines Actors. Hier wird die Logik implementiert, die ausgeführt werden soll, sobald der Actor eine Nachricht eines bestimmten Typs erhält
- erhält der Actor eine Nachricht des typen `ExecPrompt` wird ein `userInput`(der Log) und ein `systemInput` (der Befehl was mit dem Log zu tun ist) an OpenAI gegeben und die Rückgabe dessen wird dem Absender (`sender()`) der ursprünglichen Nachricht  geschickt.

Das Modul `import akkaSystem.actorActions.OpenAIPromptActorActions._` ist gar nicht so spannend. Hier passieren drei Dinge:
1. Schicke einen Request an OpenAI
2. *warte* auf die Antwort von OpenAI
3. gib die Textantwort von OpenAI zurück

Den Code hierzu halte ich mal "hinter dem Berg". Bei Bedarf wirf einfach ein Blick ins Open Source Projekt.

### LLM Actor
Der LLM Actor koordiniert in erster Linie den Nachrichtendurchlauf. Ich befürchte die Logik ist hier nicht ganz sauber strukturiert. Ich wollte aber auch nicht zu kleinteilig vorgehen ... na ja. Schau selbst:

```scala
class LLMActor(tagRepositoryActor: ActorRef, openAIPromptActor: ActorRef) extends Actor {
  import akkaSystem.actorActions.LLMActorActions._

  def parseLineSplitLLMResponse(text: String): Array[String] = {
    val lineSplit = text.split("\n").filter(_.nonEmpty)

    lineSplit match {
      case lines if lines.forall(_.startsWith("- ")) => lines.map(_.substring(2))
      case lines if lines.forall(_.startsWith("-")) => lines.map(_.substring(1))
      case _ => lineSplit
    }
  }

  override def receive: Receive = {
    case ExtractTechStack(logId, text) => openAIPromptActor ! ExecPrompt(logId, text, Prompts.extractTechStack)
    case ExtractTechStackResponse(log, llm_response) => tagRepositoryActor ! BulkCreate(log, parseLineSplitLLMResponse(llm_response))
    case _ => println("Not supported Action")
  }

}
```
- der `LLMActor` bekommt zwei `ActorRef` Objekte übergeben. Das sind jeweils zwei Actoren, an die Nachrichten geschickt werden sollen. Vermutlich wäre es sogar sauberer hier mit einer vernünftigen Hierarchie zu arbeiten und den `LLMActor` selbst diese Actoren erstellen zu lassen
- `import akkaSystem.actorActions.LLMActorActions._` sind die definierten Aktionen bzw. Typen die Nachrichten haben können, die anschließend zu einer Aktion führen
- `parseLineSplitLLMResponse` diese Methode versucht die hoffentlich Zeilenseparierte Rückgabe von OpenAI in eine Liste von Tags zu bringen
- die `receive` Methode ist das Herzstück eines Actors. Hier wird die Logik implementiert, die ausgeführt werden soll, sobald der Actor eine Nachricht eines bestimmten Typs erhält
- erhält der Actor eine Nachricht des Typs `ExtractTechStack`, wird er eine Nachricht an den `openAIPromptActor` schicken. Diese Nachricht beinhaltet den User-Log sowie den Prompt der OpenAI hinterher den Kontext geben wird, was zu tun ist
- erhält der Actor eine Nachricht des Typs `ExtractTechStackResponse` wird ebenfalls eine Nachricht an den `tagRepositoryActor` geschickt. In dem Fall beinhaltet die Nachricht die `id` eines Logs sowie die Liste an extrahierten Tags.

Anbei noch der System Prompt:
```scala
case object Prompts {
  val extractTechStack: String =
    """
      | You'll get a text about what a person done. The person might be a developer, datascientist,
      | data engineer or something different in that area.
      |
      | Please extract only the tech-stack from the text without interpreting any meaning.
      | please response in a list of strings with format `- TAG_VALUE` and put each TAG in it's own
      | line.
      |""".stripMargin
}
```

### TagRepository Actor
Der Tag Repository Actor kümmert sich darum, dass die extrahierten Tags ihren Weg in die Datenbank finden. Das Repository dazu hast du an dieser Stelle zwar noch nicht implementiert, aber der `trait` reicht schon aus, um den Code zu schreiben. Das Zeigt wie sinnvoll es ist mit Interfaces zu arbeiten. Sie machen den Code einfach planbar.

```scala
class TagRepositoryActor(repository: TagRepository) extends Actor {  
  import akkaSystem.actorActions.TagRepositoryActorActions._  
  
  override def receive: Receive = {  
    case BulkCreate(log, tags) => repository.bulkCreate(log, tags, "tech-stack")  
    case _ => println("Not supported Action")  
  }  
}
```

Das ist der vermutlich einfachste Actor. Ich spare mir an dieser Stelle jede weitere Erklärung. Ich denke, dass hier wenig neues passiert.

### Tag Repository
Bislang gibt es nur eine abstrakte Klasse des Tag Repositories, die Implementierung fehlt noch. Das wird jetzt nachgeholt.
```scala
class TagImp @Inject()(db: Database) extends TagRepository {
  def bulkCreate(log: Long, tags: Array[String], typeOf: String): Try[Array[Tag]] = Try {
    implicit val parser: RowParser[Tag] = Macro.namedParser[Tag](ColumnNaming.SnakeCase)
    db.withConnection{ implicit conn =>
      SQL(
        """
          |WITH ins AS (
          |  INSERT INTO tags(log, type_of, value)
          |  VALUES ({log}, {type_of}, UNNEST({tagsArray}))
          |  RETURNING id, log, type_of, value
          |)
          |SELECT * FROM ins;
        """.stripMargin
      ).on(
        "log" -> log,
        "type_of" -> typeOf,
        "tagsArray" -> tags
      ).as(parser.*).toArray
    }
  }
}
```
- das SQL Statement beinhaltet ein `UNNEST`, dabei wird ein Array in einzelne Rows aufgeteilt, ein cooles Feature der Postgres Datenbank!
- Die Rückgabe wird in eine Liste von Tags geparsed. Dies passiert einerseits über das `RETURNING` im Statement, andererseits weiter unten über den `.as(parser.*).toArray` bei dem die Rückgabe zu einem Array an Tag Objekten geparsed wird. Weiter oben im Code gibt es eine als implizit deklarierte Variable `implicit val parser: RowParser[Tag] = Macro.namedParser[Tag]` auf die im parser zugegriffen wird, sodass der Parser weiß, welche Objekte instantiiert werden sollen.


### Actor System
Innerhalb von Akka liegen die Actors in einem Actor System. Innerhalb dieses Systems gibt es einen gemeinsamen Threadpool den sie nutzen können und können miteinander kommunizieren.

```scala
@Singleton
case class ActorSystemManager @Inject()(
  db: Database,
  tagRepository: TagRepository,
)(implicit config: Configuration){
  implicit val system: ActorSystem = ActorSystem("MainSystem")
  private val tagRepositoryActor: ActorRef = system.actorOf(Props(new TagRepositoryActor(tagRepository)), "TagRepositoryActor")
  private val openAiRepositoryActor: ActorRef = system.actorOf(Props(new OpenAIPromptActor), "OpenAIPromptActor")

  val llm_actor: ActorRef = system.actorOf(RoundRobinPool(5).props(Props(new LLMActor(tagRepositoryActor, openAiRepositoryActor))), "LLMActor")
}
```
- `@Singleton` ist eine Annotation, die sicherstellt, dass von dieser Klasse nur eine Instanz im gesamten Play Framework-Anwendungskontext erstellt wird.
- `@Inject` injiziert die Abhängigkeiten `DataBase` und das `TagRepository` - hierbei wird natürlich die konkrete Implementierung übergeben und nicht das Interface. Auch wenn es etwas widersprüchlich dargestellt ist. Aber auch das passiert an anderer Stelle über ein binding: `bind(classOf[TagRepository]).to(classOf[TagImp])`
- ein `ActorSystem` wird erstellt, das eine Laufzeitumgebung für Akka-Aktoren bereitstellt. Jeder Akka-basierte Anwendung benötigt genau ein ActorSystem
- im folgenden werden zwei Actoren der Typen `TagRepositoryActor` und `OpenAIPromptActor` im Actor System instantiiert
- anschließend wird ein Actor namens `LLMActor` erstellt, und anstelle von nur einem solchen Actor wird ein Pool aus fünf solcher Actors erstellt, die mit einer "Round Robin"-Strategie Nachrichten verarbeiten. Dies bedeutet, dass Nachrichten nacheinander an jeden der fünf Actors weitergeleitet werden, wodurch die Last gleichmäßig verteilt wird.
