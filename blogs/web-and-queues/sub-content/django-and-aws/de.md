# Django, Simple Service Queue und AWS Lambda
[//]: # (START CUSTOM SCRIPT)
[//]: # (START MARKDOWNREF)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/assets/project-description/de.md)
[//]: # (END MARKDOWNREF)
[//]: # (END CUSTOM SCRIPT)

## Überblick
In dieser Lösung wird Django mit Serverless Funktionen der Amazon Web Services kombiniert.
Da es keine vollständige Liste an Technologien geben kann, wird ein LLM zur Informationsextraktion
verwendet, im konkreten GPT-3.5-Turbo von OpenAI.

Allerdings dauert die Abfrage gegen diese API verhältnismäßig lange und unter Last schlägt sie auch
gerne mal fehl. Daher soll die Nutzerinteraktion (speichere Log) von der Informationsextraktion
durch OpenAI getrennt werden.

Diese Trennung wird durch eine Teilung der Software in zwei Bestandteile sowie die Kopplung über
den Simple Queue Service ermöglicht.

Dies wird auch Microservice Architektur genannt. Zur Kommunikation mit einem Client wird Pythons 
Django Framework genutzt. Mit dem erstellen eines Log-Beitrags wird eine Nachricht an den Simple 
Queue Service (SQS) geschickt. Eine Lambda Funktion wird durch diesen Simple Queue Service getriggert.

### Die Vorteile
- Django ist ein brilliantes Webframework, mit einer riesigen Community und atemberaubend guten, generischen Funktionen und Klassen
- der Simple Queue Service erlaubt eine einfache Entkopplung der Microservices und Django kann Nachrichten a la "Fire and Forget" feuern. Sollte es zu Last Peaks kommen, können diese durch den SQS als Puffer abgefangen werden und nachträglich verarbeitet werden. Außerdem gibt es das Konzept der Dead Letter Queue. Sollte es zu Fehlern kommen (z.B. weil OpenAI nicht immer in der Form Antwortet, die wir brauchen), können die Fehlerhaften Nachrichten in die Dead Letter Queue geschoben werden und nachträglich nochmal verarbeitet werden
- Deployments sind unter der Verwendung des SQS kein Problem. Da der SQS ausgelagert ist, gehen bei einem Deployment keine Nachrichten verloren
- die Lambda Funktionen sind Serverless und bringen von Haus aus ein durchsuchbares Logging mit
- es gibt ein hervorragendes Gratisprogramm von AWS in Bezug auf die Lambda Funktionen mit bis zu 1 Millionen Anforderungen und 3,2 Millionen Millisekungen gratis pro Monat
- AWS garantiert für die Permanente Verfügbarkeit ihrer Lambda Funktionen

### Die Nachteile**
- Microservices machen es nicht immer ganz einfach von Ende zu Ende zu debuggen - es müssen immer mehrere Anwendungen gleichzeitig laufen. Bei kleinen Anwendungen (wie Dev-Guro) ist das u.U. zu umständlich
- der Simple Queue Service ist ein nativer Service in der Amazon Cloud. Solche Cloud Dienste Lokal z.B. durch Docker laufen zu lassen ist immer etwas hacky
- weder der SQS noch die Lambda Funktion hat ein striktes Schema für die Entitäten. Spätestens in der Lambda Funktion muss das Nachrichtenformat streng genommen nochmal extra validiert werden
- es muss für die Rückgabe der Lambda Funktion nach Django ein extra Endpunkt bereitgestellt werden


## Theoretisches und Konzepte
In diesem Projekt wird AWS verwendet. AWS ist bekanntlich einer der drei großen Clound Anbieter 
(Amazon Web Services, Microsoft Azure, Google Cloud Console) - vermutlich ist AWS sogar der Größte 
der drei Anbieter.

Die angewendeten Services sind in ähnlicher Form aber in jedem der drei großen Anbieter verfügbar. 
Im folgenden wird sich also auf sehr generelle Aspekte beschränkt, die in diesem Projekt Anwendung finden:


- Serverless Functions
- Message Systems und Queues


### Serverless Functions
Serverless ist ein Cloud-Computing-Ausführungsmodell, bei dem der Cloud-Anbieter die Infrastruktur verwaltet, 
auf der Code ausgeführt wird. Man könnte sagen, dass der Name "serverless" ein wenig irreführend ist, da immer 
noch Server im Spiel sind. Der Unterschied besteht jedoch darin, dass Entwickler und Entwicklerinnen sich nicht 
um die Server-Verwaltung kümmern müssen - der Cloud Anbieter garantiert für Aktualität und Lauffähigkeit.

Serverless Functions sind eine spezielle Art von Serverless-Diensten, bei denen Entwickler Einzelfunktionen 
bereitstellen, die in Reaktion auf bestimmte Ereignisse (z. B. eine HTTP-Anfrage, eine neue File im Blob Storage, 
eine Änderung in einer Datenbank, uvm.) ausgeführt werden - das wird übrigens *ereignisgesteuerte Ausführung* 
genannt. Hierzu gibt es bei mir bereits [einen Artikel hier](https://jangascodingplace.com/blog/article/event-driven-programming-mit-serverless-functions).

In diesem Projekt werden die Lambda Funktionen von AWS genutzt.


### Queues und Message Systeme
In Microservice-Architekturen sind die Dienste oft so konzipiert, dass sie unabhängig und isoliert voneinander 
arbeiten. Um eine effiziente und entkoppelte Kommunikation zwischen diesen Diensten zu gewährleisten, werden häufig 
Warteschlangen (Queues) und Nachrichtensysteme (Message Systems) eingesetzt. 

Eine Queue ist die Implementierung einer Datenstruktur, die Nachrichten oder Aufgaben in der Reihenfolge ihrer 
Ankunft ablegt. In einer Microservice-Architektur kann ein Dienst eine Nachricht in eine Warteschlange senden, und 
ein anderer Dienst kann diese Nachricht später abholen und verarbeiten.

Dadurch wird eine Entkopplung ermöglicht. Dienste müssen nicht gleichzeitig online oder verfügbar sein. Auch 
Elastizität wird geboten. Wenn ein Dienst plötzlich eine große Anzahl von Anfragen erhält, können diese Anfragen in 
eine Warteschlange gestellt und schrittweise verarbeitet werden.

In diesem Projekt wird der Simple Queue Service von AWS genutzt.


## Die Implementierung

### Models
Fangen wir mit den Models an. Davon gibt es zwei: `Log` und `Tag`. Der Einfachheit halber lasse ich die Nutzer Zuordnung im folgenden mal weg. Um das klar zu stellen: das Usermanagement in Django ist erstaunlich einfach. Nur möchte ich den Fokus eher auf anderes lenken. Zu diesem Punkt komme ich aber später nochmal in der Conclusion.
```python
# models.py

from django.db import models  
  
  
class Log(models.Model):  
    entry = models.TextField()  
    timestamp = models.DateTimeField(auto_now_add=True)  
  
  
class Tag(models.Model):  
    type = models.CharField(  
        choices=[  
            ("tech-stack", "Tech Stack"),  
        ],  
        max_length=20,  
    )  
    value = models.CharField(  
        max_length=120,  
    )  
    log = models.ForeignKey(  
        Log,  
        on_delete=models.CASCADE,  
        related_name="tags",  
    )  
    timestamp = models.DateTimeField(  
        auto_now_add=True,  
    )
```
Viel gibt es hier nicht zu sagen, aber hier doch noch ein paar Eckdaten
- `auto_now_add=True` bedeutet, dass ohne weiteres zutun beim Erstellen des Objekts der Zeitstempel dem aktuellen Zeitstempel entspricht
- im Model `Tag` und Feld `type` ist ein `choice_field` . Dort werden im Prinzip alle erlaubten Charakter-Werte definiert
- der `related_name="tags"` im Foreignkey erlaubt ein wenig syntactic sugar und ermöglicht einen schnelleren Zugriff auf die Objekte die auf einen bestimmten Tag zeigen. z.B. so: `Log.objects.get(id=1).tags` anstelle von `Tag.objects.filter(log__id=1)`

### Signals
Nachdem ein Log erstellt wurde, soll dieser an den Simple Queue Service gepublished werden. Dies tue ich im folgenden mit einem `post_save` Signal.

```python
# signals.py

import boto3  
import json  
  
from django.db.models.signals import post_save  
from django.dispatch import receiver  
from django.conf import settings  
  
from . import models  
  
  
@receiver(post_save, sender=models.Log)  
def pub_log_event_to_sqs(instance: models.Log, created: bool, **kwargs):  
    if not created:  
        return  
  
    event_data = {  
        "id": instance.id,  
        "entry": instance.entry,  
    }  
  
    sqs = boto3.client(  
        "sqs",  
        region_name=settings.AWS_S3_REGION_NAME,  
        aws_access_key_id=settings.AWS_ACCESS_KEY_ID,  
        aws_secret_access_key=settings.AWS_SECRET_ACCESS_KEY,  
    )  
    sqs.send_message(  
        QueueUrl=f"{settings.AWS_QUEUE_BASE_URL}/logs",  
        MessageBody=json.dumps(event_data),  
    )
```
- `post_save`  ist ein Signal welches nachdem ein Eintrag `Log` gespeichert wurde (sei es durch ein create oder ein update) ausgeführt wurde
- innerhalb der Methode wird sicher gegangen, dass das Objekt frisch erstellt wurde und es sich nicht um ein update handelt
- `boto3` ist das AWS SDK. Dort gibt es viele verschiedene APIs. In unserem Fall wird der `sqs` genutzt.
- Anschließend wird eine Nachricht an den SQS geschickt. Hier ist wichtig, dass die Queue bereits existieren muss

### Serializers
Ich arbeite fast ausschließlich mit den Class-Based-Views. Die Serializers helfen dort massiv den Code zu reduzieren indem der User-Request automatisch zum Objekt geparsed (serializiert) und der Response automatisch zum JSON Objekt geparsed (deserializiert) wird.

```python
# serializers.py

from rest_framework import serializers  
from . import models  
  
  
class LogSerializer(serializers.ModelSerializer):  
    class Meta:  
        model = models.Log  
        fields = "__all__"  
  
  
class TagSerializer(serializers.ModelSerializer):  
    log = serializers.PrimaryKeyRelatedField(  
        queryset=models.Log.objects,
    )  
    class Meta:  
        model = models.Tag  
        fields = "__all__"
```

- Im Falle des LogSerializers wird nur `entry` im Payload erwartet. Zurück gegeben werden dann die Felder id, entry und timestamp.
- Im Falle des `TagSerializers` funktioniert das analog. Erwartet werden dort die Felder type, value und log. Zurückgegeben wird noch zusätzlich die id und der timestamp.
	- das Feld `log` wird hier nochmal etwas angepasst. Das `PrimaryKeyRelatedField` wird verwendet, um direkt den passenden Log zu einem mitgegebenen Primary Key zu finden
	- die Verwendung des `PrimaryKeyRelatedField` wird insbesondere dann sinnvoll, wenn man vermeiden möchte, dass die Objekte auf die ein Foreign Key zeigt, mit erstellt werden. Das ist manchmal etwas nervig.


### Views

Nun fehlen im Prinzip nur noch die Views. Im konkreten Fall gibt es zwei Views: einen View zum erstellen eines Logs (für den Nutzer), sowie eines weiteren Views zum erstellen der Tags für einen Log (für die Lambda Funktion).

```python
# views.py

from rest_framework.generics import CreateAPIView  
from .serializers import LogSerializer, TagSerializer  
from .models import Log, Tag  
  
class LogCreateView(CreateAPIView):  
    queryset = Log.objects  
    serializer_class = LogSerializer  
  
  
class TagCreateView(CreateAPIView):  
    queryset = Tag.objects  
    serializer_class = TagSerializer  
  
    def get_serializer(self, *args, **kwargs):  
        serializer_class = self.get_serializer_class()  
        kwargs.setdefault('context', self.get_serializer_context())  
        return serializer_class(many=True, *args, **kwargs)
```
- `LogCreateView` erlaubt lediglich einen `post` request mit den im Serializer angegebenen Feldern. Anschließend wird ein Tag-Objekt erstellt und das erstellte Objekt an den Client zurück gegeben
- `TagCreateView` erlaubt lediglich einen `post` request mit den im Serializer angegebenen Feldern.  Anschließend wird ein Tag-Objekt erstellt und das erstellte Objekt an den Client zurück gegeben. Eine wertvolle Randnotiz ist noch, dass die Methode `get_serializer` überschrieben wurde um so zu ermöglichen, dass nicht nur ein Eintrag im Payload mitgeschickt wird, sondern eine Liste von Einträgen.
	- Diese Implementierung ist gut fürs Prototyping aber schlecht in der Performance. Es ist zu erwarten, dass ausschließlich Tags zu einem bestimmten Log mitgeschickt werden. In der aktuellen Implementierung wird der Log zu einem Primary Key allerdings nicht nur einmal, sondern n-Mal aus der Datenbank geholt. Diese Abfrage ist recht performant (da ein Primary Key abgefragt wird) und je nach Datenbank gibt es auch ein automatisches Caching. Allerdings liegt hier immenses Optimierungspotential.


### AWS Lambda

Nun gehen wir raus aus der Django Welt und rein in AWS Lambda. Für die Lambda Funktion gibt es dabei einige Hilfskonstanten sowie Methoden. Angefangen beim Prompt:

```python
PROMPT = """
    You'll get a text about what a person done. The person might be a developer, datascientist,
    data engineer or something different in that area.

    Please extract only the tech-stack from the text without interpreting any meaning.
    please response in a list of strings with format `- TAG_VALUE` and put each TAG in it's own
    line.
"""
```

Mit diesem Prompt werden OpenAI zum einen Informationen zum Kontext und zum anderen Anforderungen an die Rückgabe gegeben.
Die Herausforderung bei dieser Art Nutzung von OpenAI besteht übrigens darin, dass eine unstrukturierte Rückgabe in eine strukturierte verwandelt werden muss. Bei derartigen Abfragen gibt es keine 100% Genauigkeit der korrekten Rückgabe.

In dem Sinne bekommt die Lambda Funktion noch eine Cleanup Methode.
```python
def parse_line_split_llm_response(text: str) -> list[str]:
    line_split = [line for line in text.split("\n") if line]

    if all([line.startswith("- ") for line in line_split]):
        line_split = [line[1:] for line in line_split]

    if all([line.startswith("-") for line in line_split]):
        line_split = [line[1:] for line in line_split]

    return line_split
```

Die Antwort von OpenAI wird ein zusammenhängender Text sein. Diese Funktion wird dazu genutzt den zusammenhängenden Text in eine Liste von Strings zu teilen.

Jetzt kümmern wir uns noch über eine Methode, die einen API Request gegen OpenAI feuert.
```python
def send_open_api_request(text: str, system: str) -> str:
    data = {
        "model": "gpt-3.5-turbo",
        "messages": [
            {"role": "system", "content": system},
            {"role": "user", "content": text},
        ],
    }
    response = requests.post(
        "https://api.openai.com/v1/chat/completions",
        json=data,
        headers={"Authorization": f"Bearer {os.getenv('OPENAI_SECRET_KEY')}"},
    )
    response.raise_for_status()
    return response.json()["choices"][0]["message"]["content"]
```

An der Stelle sei gesagt, dass OpenAI auch eine Python Library anbietet. Ich habe mich hier für die Verwendung von requests entschieden - Dependeny Minimalismus.
Man könnte sich jetzt auch darüber streiten, ob die Methode weiter aufgeteilt werden sollte ... für dieses Beispiel soll es aber so genügen.

Jetzt fehlt nur noch die Lambda Funktion, die die einzelnen Stücke zusammensetzt:
```python
def lambda_handler(event, context):
    for record in event["Records"]:
        data = json.loads(record["body"])
        open_ai_answer = send_open_api_request(data["entry"], PROMPT)
        tags = parse_line_split_llm_response(open_ai_answer)
        payload = [{"log": data["id"], "value": tag, "type": "tech-stack"} for tag in tags]
        requests.post(
            f"{os.getenv('BASE_URL')}/logs/tags",
            json=payload,
        )
```
- Diese Lambda Funktion bekommt vom Simple Queue Service einzelne Events.
- diese Events werden zu einem Dictionary geparsed
- der Text des Log Events wird zusammen mit den Anforderungen an OpenAI geschickt
- die Rückgabe von OpenAI wird geparsed - dieser Teil ist extrem Fehleranfällig! OpenAI kann in allen Formaten antworten. In der Praxis passiert das leider auch
- anschließend werden die Tags via REST API call an die Django Anwendung gesendet.
