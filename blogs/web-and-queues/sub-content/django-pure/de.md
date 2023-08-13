# Ein Projekt zu Web und Queues mit Django
[//]: # (START CUSTOM SCRIPT)
[//]: # (START MARKDOWNREF)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/assets/project-description/de.md)
[//]: # (END MARKDOWNREF)
[//]: # (END CUSTOM SCRIPT)




## Überblick
In dieser Lösung wird Python Django verwendet. Da es keine vollständige Liste an Technologien 
geben kann, wird ein LLM zur Informationsextraktion verwendet, im konkreten GPT-3.5-Turbo von OpenAI.

Allerdings dauert die Abfrage gegen diese API verhältnismäßig lange und unter Last schlägt sie auch
gerne mal fehl. Daher soll die Nutzerinteraktion (speichere Log) von der Informationsextraktion
durch OpenAI getrennt werden.

Diese Trennung wird durch einen separaten Prozess auf einem eigenen Thread. Dieser Thread ist mit der
Hauptanwendung über eine Queue verbunden.



### Die Vorteile
- kein unnötiges Auslagern einzelner Funktionen
- Ende zu Ende tests sind ohne viel Aufwand machbar
- Reduktion des Codes im Vergleich einer Microservice getriebenen Lösung



### Die Nachteile
- es muss ein extra Thread zum konsumieren der Queue bereit gestellt werden. Meiner persönlichen Erfahrung nach ist es nicht immer ganz einfach einen Thread am leben zu halten / beim Fehlschlagen neu zu starten




## Theoretisches und Konzepte
Django ist ein Webframework der Programmiersprache Python. Es folgt insbesondere im Restframework dem 
Pattern des Model-View-Controller (ohne Rest-Framework ist es eher ein Model-Template-Controller). 
Innerhalb dieses Beitrags werden verschiedene Funktionen und Konzepte genutzt:
- models
- signals
- views (restframework)
- serializers (restframework)
- Class based Views (restframework)
- Model View Controller (MVC) - tbd



### Models
Ein Model in Django ist im weitesten Sinne die Repräsentation einer Tabelle. Im folgenden ein Beispiel 
für die Tabelle `Logs`

```python
from django.db import models  
  
  
class Log(models.Model):  
    entry = models.TextField()  
    timestamp = models.DateTimeField(auto_now_add=True) 
```
Die Tabelle wird in der Datenbank folgende Spalten haben:
- `id`: serial
- `entry`: varchar
- `timestamp` timestamp

Zudem bekommt das Model noch den Modelmanager vererbt und verfügt über einige Grundfunktionen. 
Hier mal ein paar Beispiele:
```python
Log.objects.get(id=1)
Log.objects.filter(timestamp__gte='2023-01-01')
Log.objects.create(entry='wrote a programm by using Python Django')
```

Die Models sind gewissermaßen Djangos Möglichkeit mit der Datenbank zu kommunizieren und Python
in die Datenbanksprache zu übersetzen, sowie umgekehrt.



### Signals
Django verfügt über einen globalen Event Loop. Es gibt verschiedene Arten von Signalen:
- Model-bezogene Signale
- Management-bezogene Signale
- Request-Bezogene Signale

So kann eine Funktion ausgeführt werden, nachdem ein Model gespeichert wurde (also nachdem etwas 
in die Datenbank geschrieben wurde), ohne diese Methode explizit aufzurufen.

Ein Signal wartet entsprechend einfach nur auf das Ausführungssignal - gewissermaßen Djangos 
Implementierung einer Event-getriebenen Logik.

Wichtig zu beachten ist dabei, dass die Signale synchron ausgeführt werden. Der Request



### Views
Ein View in Django ist der Punkt, an dem die Anfrage des Clients in die Djangowelt getragen wird 
und umgekehrt. Jeder Pfad in der URL bekommt genau einen View zugeteilt.

Die Views in Django sind extrem mächtig. So können z.B. ohne viel Code sofort Permissions geprüft 
werden, welche Art Request angenommen wird, uvm.

Django bietet dabei eine funktionale Implementierung der Views (Function Based Views), sowie eine 
Objektorientierte Implementierung (Class Based Views). Je nach persönlicher Präferenz wird dabei 
also arg unterschiedlich mit den Django views gearbeitet.



### Serializers
Das Konzept der Serializers ist elementar im Django Rest Framework. Im klassischen Django Framework 
gibt es die Forms. Im Restframework machen die allerdings nur bedingt Sinn. Dafür gibt es die Serializer.

Ein Serializer kümmert sich sowohl um die Serializierung sowie Deserializierung. Ein Modelserializer 
tut dies im konkreten Bezug auf ein Model.

```python
from rest_framework import serializers
from .models import Foo, Bar


class FooSerializer(serializers.ModelSerializer):
    class Meta:
        model = Foo
        fields = '__all__'


class BarSerializer(serializers.ModelSerializer):
    custom_1 = serializers.CharField(max_length=30, write_only=True)
    class Meta:
        model = Bar
        fields = ["id", "col_1", "custom_1"]
        extra_kwargs = {"id": {"read_only": True}}
```

Das waren einmal zwei Beispiele für einen ModelSerializer. Im ersten Fall war ich etwas faul. Hier 
erwartet der Serializer zum erstellen eines Objekts alle relevanten Eigenschaften und gibt absolut alle 
Eigenschaften zurück.

Der zweite Serializer ist da schon wesentlich expliziter. Im Fall eines creates erwartet er die Eigenschaften 
`col_1` und `custom_1` - wobei `custom_1` dabei noch nichtmal zwingend eine Eigenschaft des Models sein muss. 
Deserializieren tut die Klasse dann allerdings die Fleder `id` und `col_1`.

Insbesondere bei der Arbeit mit Class Based Views ist die Verwendung der Serializers elementar.

### Class Based Views
Insbesondere bei der Interaktion meiner Models bevorzuge ich die Class Based Views. Innerhalb der Class 
Based Views kann mit extrem wenig Code viel Funktionalität entstehen.

```python
from rest_framework.generics import RetrieveAPIView
from rest_framework.permissions import IsAuthenticated
from rest_framework.authentication import TokenAuthentication
from .serializers import LogSerializer
from .models import Log

class LogCreateView(RetrieveAPIView):
    permission_classes = (IsAuthenticated,)
    authentication_classes = (TokenAuthentication,)
    queryset = Log.objects
    serializer_class = LogSerializer
```
Dieser View prüft ob ein User eingeloggt ist (über den Accesstoken), validiert automatisch den Payload des 
requests (über den Serializer) und antwortet, sucht in der Datenbank über den Primary Key direkt nach einem 
passenden Eintrag und antwortet mit einem festen JSON Format dem Client und dabei wurden noch nichtmal alle 
Möglichkeiten des Views voll ausgeschöpft.

Die Class-Based-Views im Django Restframework sind exzellent geschrieben und dadurch extrem flexibel. Durch 
geschicktes überschreiben einzelner Methoden, kann massiv Code gespart werden.




## Die Implementierung



### Models
Die Models sind analog zum vorigen Beispiel. Den Code füge ich hier nochmal ein, die Erklärung lasse ich aber mal unter den Tisch fallen:
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


### Serializers
```python
# serializers.py

from rest_framework import serializers  
from . import models  
  
  
class LogSerializer(serializers.ModelSerializer):  
    class Meta:  
        model = models.Log  
        fields = "__all__"  
```
Im Prinzip ist das ein Standard Model Serializer. Es werden alle Eigenschaften des Models `Log` erwartet, die erforderlich sind, um ihn in die Datenbank zu schreiben (im aktuellen Fall ist es lediglich `entry` ) und es werden alle Eigenschaften zurück gegeben, die das Model beschreiben (im aktuellen Fall `id`, `entry`, `timestamp`



### Views
```python
# views.py

from rest_framework.generics import CreateAPIView  
from .serializers import LogSerializer
from .models import Log
  
class LogCreateView(CreateAPIView):  
    queryset = Log.objects  
    serializer_class = LogSerializer
```

Es gibt lediglich einen Serializer, analog dazu gibt es auch lediglich einen view. Dieser View sorgt dafür dass nach einem eingehenden Request ein Log-Objekt in die Datenbank geschrieben wird.

Ich finde es nach wie vor atemberaubend, mit wie wenig Aufwand neue API-Endpunkte entstehen können. Der `CreateAPIView` vererbt eine Vielzahl an Methoden. Ich werde nicht müde jeder Person zu empfehlen sich ganz konkret den Code der Generic Views des Django Restframeworks mal anzusehen. Meiner Meinung nach ein Meisterwerk!



### Queues
So ... es wird wild. Im Beispiel vorher wurden Nachrichten an den Simple Queue Service geschickt. Dieser wird hier nun abgelöst durch eine Python Queue.

Die Queues werden zentral definiert.

```python
# queues.py

from queue import Queue


create_tag_for_log_queue = Queue()
```
Hier wird lediglich die Queue `create_tag_for_log_queue` initialisiert. Wichtig zu beachten ist, dass nicht automatisch alle Nachrichten in der Queue persistiert werden. Das kann für Downtimes kritisch sein, da u.U. Nachrichten verloren gehen.



### Signals
Nachdem ein Log erstellt wurde, soll dieser an die Python Queue gesendet werden. Dies geschieht im folgenden mit einem `post_save` Signal.

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from configs import queues

from . import models


@receiver(post_save, sender=models.Log)
def pub_log_event_to_sqs(instance: models.Log, created: bool, **kwargs):
    if not created:
        return

    queues.create_tag_for_log_queue.put({"id": instance.id, "entry": instance.entry})
```
- `post_save`  ist ein Signal welches nachdem ein Eintrag `Log` gespeichert wurde (sei es durch ein create oder ein update) ausgeführt wurde
- innerhalb der Methode wird sicher gegangen, dass das Objekt frisch erstellt wurde und es sich nicht um ein update handelt
- nachdem sicher gegangen wurde, dass es sich bei dem Signal um eine `create` Operation gehandelt hat, wird eine Nachricht in Form eines Dictionaries an die Python Queue gesendet



### Queue Consumer
Nun wird noch ein Prozess benötigt, der die queue konsumiert. Wichtig hier ist, dass er in einem eigenen Thread ausgeführt wird.

```python
# consumer.py
from configs import queues
from .models import Tag, Log

# other imports
# PROMPT = ...
# def parse_line_split_llm_response(): ...
# def send_open_api_request(): ...

def consume_tag_create_queue():
    while True:
        data = queues.create_tag_for_log_queue.get()
        try:
            open_ai_answer = send_open_api_request(data["entry"], PROMPT)
            tags = parse_line_split_llm_response(open_ai_answer)
            log = Log.objects.get(id=data["id"])
            for tag in tags:
                Tag.objects.create(log=log, value=tag, type="tech-stack")
        except Exception as e:
            print(e)
        finally:
            queues.create_tag_for_log_queue.task_done()
```
- die Konstante `PROMPT` sowie die Methoden `parse_line_split_llm_response` und `send_open_api_request` sind absolut identisch zu den Methoden des vorigen Beispiels und werden daher nicht weiter erläutert
- die `consume_tag_create_queue` beinhaltet eine Endlosschleife, die die im Schritt zuvor definierte queue `create_tag_for_log_queue` konsumiert.
- Die empfangene Nachricht ist ein Dictionary. Theoretisch können auch Objektinstanzen in die Queue geschrieben und von ihr gelesen werden. Sollte es jedoch zur Persistierung der Queue kommen, wird hier noch ein Serializer benötigt.
- Der Log-Entry wird mit der Systemanweisung an OpenAI gesendet
- die Rückgabe von OpenAI wird zu einer Liste geparsed - in der Hoffnung, dass alle Tags mindestens zeilensepariert zurück gegeben werden
- anschließend wird der Log Eintrag zur Nachricht aus der Datenbank geholt. Dies wäre vermeidbar, wenn der Eintrag als Instanz anstelle des Dictionaries gegeben würde
- zuletzt wird jeder Tag in die Datenbank geschrieben
- Egal ob der Prozess Erfolgreich war oder nicht, die Nachricht wird als erledigt markiert



### Apps
Im Schritt zuvor wurde eine Funktion mit einer `while True` Schleife implementiert. Diese soll in einem eigenen Thread ausgeführt werden (sonst würde sie das ganze Programm blockieren). Dies passiert im app - Modul.

```python
# apps.py

from threading import Thread
from django.apps import AppConfig


class LogsConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "Logs"

    def ready(self):
        from . import signals  # noqa
        from . import consumer
        Thread(target=consumer.consume_tag_create_queue, daemon=True).start()
```
Diese Datei wird standardmäßig beim Erstellen einer App erzeugt. Hinzugefügt wurde lediglich die Methode `ready`. Django unterstützt an einigen Stellen das Lazy Loading - d.h. Objekte werden im zuletzt möglichen Moment geladen. Das trifft insbesondere auf die models zu. Sobald die App inklusive models geladen ist, wird die ready Methode ausgeführt.

Die ready Methode macht dabei zwei Dinge:
1. registrieren des `post_save` Signals durch einfaches importieren des Moduls
2. Starten des consumer-threads mit der endlosen while Schleife



### Alternativen
Ich bin kein großer Freund von Threads in Python. Ich habe immer sehr viel Aufwand betrieben, um einen Thread am leben zu halten und ihn im Zweifel neu zu starten ist auch nicht immer ganz einfach.

Django bietet ein weiteres signal `request_finished`. Sobald eine Rückgabe von irgendeinem View erfolgte, wird ein `request_finished`Signal erzeugt. Damit bekommt der Client sofort seine Antwort und erst nachdem er die Antwort bekam, wird der API-Call an OpenAI ausgeführt.
Allerdings sind die Signale in Django selbst synchron. Dieses Signal blockiert also unter Umständen andere Anfragen. Das finde ich sehr Schade! Allgemein supported Django in Version 4.x async views, Ich finde es schade, dass dies aktuell noch nicht auf die Signale zutrifft - zumindest habe ich keine einfache Lösung gefunden.

Eine weitere Möglichkeit könnte in der Verwendung von Celery bestehen. Celery ist ein wenig Django's Antwort auf die Frage wie automatisiert Tasks ausgeführt werden können. Theoretisch könnte anstelle einer Nachricht an eine Queue eine Task für Celery erstellt werden. Diese Variante halte ich tatsächlich für ein wenig geeigneter als die hier aufgeführte Lösung. Es ist nunmal leichter Tasks auszuführen als einen Thread dauerhaft am leben zu halten. Und Celery ist in der Lage eine Vielzahl von Tasks abzuarbeiten. Der Grund warum ich Celery nicht als expliziten Lösungsansatz vorgestellt habe, ist das komplexere Setup. Dev-Guro läuft auf Amazon Elastic Container Services Fargate. Ein serverloser Ansatz bei dem sich Amazon um Deployments, Monitoring uvm. kümmert. Durch Celery hätte es mindestens zwei Jobs für ECS benötigt. Zudem muss überlegt werden, wie Celery von den Jobs erfährt. Das würde vermutlich über das konsumieren eines Simple Queue Services geschehen. Ein gewisser verstärkter operativer Aufwand durch die Verwendung von Celery ist also nicht ganz zu vermeiden.
