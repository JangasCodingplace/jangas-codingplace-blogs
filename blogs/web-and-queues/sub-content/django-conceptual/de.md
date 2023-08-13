# Django
Django ist ein Webframework der Programmiersprache Python. Es folgt insbesondere im Restframework dem Pattern des Model-View-Controller (ohne Rest-Framework ist es eher ein Model-Template-Controller). Innerhalb dieses Beitrags werden verschiedene Funktionen und Konzepte genutzt:
- models
- signals
- views (restframework)
- serializers (restframework)
- Class based Views (restframework)
- Model View Controller (MVC) - tbd

## Models
Ein Model in Django ist im weitesten Sinne die Repräsentation einer Tabelle. Im folgenden ein Beispiel für die Tabelle `Logs`

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

Zudem bekommt das Model noch den Modelmanager vererbt und verfügt über einige Grundfunktionen. Hier mal ein paar Beispiele:
```python
Log.objects.get(id=1)
Log.objects.filter(timestamp__gte='2023-01-01')
Log.objects.create(entry='wrote a programm by using Python Django')
```

Die Models sind gewissermaßen Djangos Möglichkeit mit der Datenbank zu kommunizieren und Python in die Datenbanksprache zu übersetzen, sowie umgekehrt.

## Signals
Django verfügt über einen globalen Event Loop. Es gibt verschiedene Arten von Signalen:
- Model-bezogene Signale
- Management-bezogene Signale
- Request-Bezogene Signale

So kann eine Funktion ausgeführt werden, nachdem ein Model gespeichert wurde (also nachdem etwas in die Datenbank geschrieben wurde), ohne diese Methode explizit aufzurufen.

Ein Signal wartet entsprechend einfach nur auf das Ausführungssignal - gewissermaßen Djangos Implementierung einer Event-getriebenen Logik.

Wichtig zu beachten ist dabei, dass die Signale synchron ausgeführt werden. Der Request

## Views
Ein View in Django ist der Punkt, an dem die Anfrage des Clients in die Djangowelt getragen wird und umgekehrt. Jeder Pfad in der URL bekommt genau einen View zugeteilt.

Die Views in Django sind extrem mächtig. So können z.B. ohne viel Code sofort Permissions geprüft werden, welche Art Request angenommen wird, uvm.

Django bietet dabei eine funktionale Implementierung der Views (Function Based Views), sowie eine Objektorientierte Implementierung (Class Based Views).
Je nach persönlicher Präferenz wird dabei also arg unterschiedlich mit den Django views gearbeitet.

## Serializers
Das Konzept der Serializers ist elementar im Django Rest Framework. Im klassischen Django Framework gibt es die Forms. Im Restframework machen die allerdings nur bedingt Sinn. Dafür gibt es die Serializer.

Ein Serializer kümmert sich sowohl um die Serializierung sowie Deserializierung. Ein Modelserializer tut dies im konkreten Bezug auf ein Model.

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

Das waren einmal zwei Beispiele für einen ModelSerializer. Im ersten Fall war ich etwas faul. Hier erwartet der Serializer zum erstellen eines Objekts alle relevanten Eigenschaften und gibt absolut alle Eigenschaften zurück.

Der zweite Serializer ist da schon wesentlich expliziter. Im Fall eines creates erwartet er die Eigenschaften `col_1` und `custom_1` - wobei `custom_1` dabei noch nichtmal zwingend eine Eigenschaft des Models sein muss. Deserializieren tut die Klasse dann allerdings die Fleder `id` und `col_1`.

Insbesondere bei der Arbeit mit Class Based Views ist die Verwendung der Serializers elementar.

## Class Based Views
Insbesondere bei der Interaktion meiner Models bevorzuge ich die Class Based Views. Innerhalb der Class Based Views kann mit extrem wenig Code viel Funktionalität entstehen.

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
Dieser View prüft ob ein User eingeloggt ist (über den Accesstoken), validiert automatisch den Payload des requests (über den Serializer) und antwortet, sucht in der Datenbank über den Primary Key direkt nach einem passenden Eintrag und antwortet mit einem festen JSON Format dem Client und dabei wurden noch nichtmal alle Möglichkeiten des Views voll ausgeschöpft.

Die Class-Based-Views im Django Restframework sind exzellent geschrieben und dadurch extrem flexibel. Durch geschicktes überschreiben einzelner Methoden, kann massiv Code gespart werden.
