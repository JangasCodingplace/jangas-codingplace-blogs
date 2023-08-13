# Die Implementierung mit Django und der Python Queue
Zugegeben, nicht besonders einfallsreich, aber manchmal sieht man den Weg vor lauter Bäumen nicht.

Es muss nicht immer zwingend ein extra Messaging Service her (z.B. SQS, RabbitMQ, Kafka), der eine entkoppelte Kommunikation zwischen einzelnen Funktionen ermöglicht. Insbesondere wenn eine Anwendung mit sich selbst kommuniziert, darf dieser Ansatz durchaus infrage gestellt werden.

Im folgenden stelle ich eine pure Django Implementierung unter der Verwendung der Python Queue aus der Standardbibliothek vor.

**Vorteile**
- kein unnötiges Auslagern einzelner Funktionen
- Ende zu Ende tests sind ohne viel Aufwand machbar
- Reduktion des Codes im Vergleich der Lösung zu vorher (z.B. wegfallen eines extra Service Endpunkts)
- Die queue kann persistiert werden, sodass nicht abgeholte Nachrichten nach einem Deployment erneut in die Queue gegeben werden können

**Nachteile**
- es muss ein extra Thread zum konsumieren der Queue bereit gestellt werden. Meiner persönlichen Erfahrung nach ist es nicht immer ganz einfach einen Thread am leben zu halten / beim Fehlschlagen neu zu starten

Auch hier verkneife ich mir die Anekdote nicht: rückblickend halte ich diesen Ansatz für meine Dev-Guro Anwendung und viele weitere Anwendungen aus der Vergangenheit für deutlich angenehmer. Letztlich wird die Queue nur genutzt um zwei Services voneinander zu entkoppeln. Andere typische Anwendungen für Queues (z.B. verschiedene Datenproduzenten oder verschiedene Datenkonsumenten) werden nicht genutzt.

Ich neige wohl zu sehr Tool-getriebenen Lösungen. Dieser Blogbeitrag soll mich langfristig dazu ermahnen, etwas zurückhaltender zu handeln.

## Models
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

## Serializers
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

## Views
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

## Queues
So ... es wird wild. Im Beispiel vorher wurden Nachrichten an den Simple Queue Service geschickt. Dieser wird hier nun abgelöst durch eine Python Queue.

Die Queues werden zentral definiert.

```python
# queues.py

from queue import Queue


create_tag_for_log_queue = Queue()
```
Hier wird lediglich die Queue `create_tag_for_log_queue` initialisiert. Wichtig zu beachten ist, dass nicht automatisch alle Nachrichten in der Queue persistiert werden. Das kann für Downtimes kritisch sein, da u.U. Nachrichten verloren gehen.

## Signals
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

## Queue Consumer
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

## Apps
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

## Alternativen
Ich bin kein großer Freund von Threads in Python. Ich habe immer sehr viel Aufwand betrieben, um einen Thread am leben zu halten und ihn im Zweifel neu zu starten ist auch nicht immer ganz einfach.

Django bietet ein weiteres signal `request_finished`. Sobald eine Rückgabe von irgendeinem View erfolgte, wird ein `request_finished`Signal erzeugt. Damit bekommt der Client sofort seine Antwort und erst nachdem er die Antwort bekam, wird der API-Call an OpenAI ausgeführt.
Allerdings sind die Signale in Django selbst synchron. Dieses Signal blockiert also unter Umständen andere Anfragen. Das finde ich sehr Schade! Allgemein supported Django in Version 4.x async views, Ich finde es schade, dass dies aktuell noch nicht auf die Signale zutrifft - zumindest habe ich keine einfache Lösung gefunden.

Eine weitere Möglichkeit könnte in der Verwendung von Celery bestehen. Celery ist ein wenig Django's Antwort auf die Frage wie automatisiert Tasks ausgeführt werden können. Theoretisch könnte anstelle einer Nachricht an eine Queue eine Task für Celery erstellt werden. Diese Variante halte ich tatsächlich für ein wenig geeigneter als die hier aufgeführte Lösung. Es ist nunmal leichter Tasks auszuführen als einen Thread dauerhaft am leben zu halten. Und Celery ist in der Lage eine Vielzahl von Tasks abzuarbeiten. Der Grund warum ich Celery nicht als expliziten Lösungsansatz vorgestellt habe, ist das komplexere Setup. Dev-Guro läuft auf Amazon Elastic Container Services Fargate. Ein serverloser Ansatz bei dem sich Amazon um Deployments, Monitoring uvm. kümmert. Durch Celery hätte es mindestens zwei Jobs für ECS benötigt. Zudem muss überlegt werden, wie Celery von den Jobs erfährt. Das würde vermutlich über das konsumieren eines Simple Queue Services geschehen. Ein gewisser verstärkter operativer Aufwand durch die Verwendung von Celery ist also nicht ganz zu vermeiden.
