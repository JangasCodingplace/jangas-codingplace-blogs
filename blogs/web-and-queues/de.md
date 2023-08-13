# Queueing und Web
Im folgenden werden drei Lösungsansätze für die selbe Aufgabenstellung vorgestellt. In diesen Ansätzen geht es darum, dass zwei Prozesse voneinander entkoppelt werden sollen:

1. eine Nutzeranfrage schreibt einen Eintrag in die Datenbank
2. mit dem neuen Eintrag wird ein "langsamer" Prozess getriggert

Die Antwort an den Nutzer oder die Nutzerin soll dabei aber möglichst sofort erfolgen und nicht extra auf die Verarbeitung des langsamen Prozesses gewartet werden.

## Über diesen Artikel
Dieser Artikel ist Teil von "JangasCodingplace" - meine persönliche Webseite. Mein Anliegen ist es nicht Tutorials zu schreiben - davon gibt es genug. Das können andere auch besser! Etwas was mir aber oft ein wenig fehlt: ganzheitliche, themenübergreifende Projekte und die Vermittlung von Konzepten. Darauf möchte ich mich ein wenig stürzen.

Zudem ist JangasCodingplace ein Ort, der mir dabei helfen soll, meine kleinen Freizeitprojekte abzuschließen und vorzustellen. Dieses Projekt ist ein Teil davon.

## Das Szenario
Dieses Projekt stellt eine Teilfunktion meiner "Dev-Guro" Anwendung dar (diese stelle ich bei Gelegenheit mal vor). In der Teilfunktion werden Dev-Logs an eine API gesendet. Anschließend wird der Techstack in Form von Tags aus den Log Einträgen gezogen. Das übernimmt OpenAI.

Hier mal ein konkretes Beispiel:
> Heute habe ich eine Anwendung mit Django geschrieben. Diese Anwendung schreibt Informationen nach Elasticsearch und ist auf ImaginaryNoneExistence Cloud deployed.

OpenAI extrahiert dann folgende Informationen:
- Django
- Elasticsearch
- ImaginaryNoneExistence Cloud

Der Techstack ist wichtig für das System und weitere Prompts. Die Nutzerinnen und Nutzer der Anwendung interessieren sich dafür aber nicht wirklich und sollen die Informationextraktion bestenfalls nicht in Form von längeren Wartezeiten merken.

## Umgang mit dieser Projektvorstellung
Ich werde drei unterschiedliche Lösungsanstätze vorstellen, ein Ansatz wilder als der andere:
- Python Django pur mit der Python Queue
- Python Django & Amazon Simple Queue Service & Amazon Lambda
- Scala Play & Akka

Dabei sind die Ansätze jeweils zweigeteilt:
1. Vorstellung der verwendeten Konzepte und Tools für einen grundsätzlichen Aufbau der Verständnis
2. Konkrete Implementierung im Code

Dabei ist zu beachten, dass dies **kein** ganzheitliches Tutorial ist. Es ist also nicht in allen Bereichen perfekt erklärt und das ist auch nicht unbedingt der Anspruch.

Dies ist ein Mehrteiliger Beitrag. Alle Lösungen funktionieren unabhängig voneinander. Entsprechend kannst du dich auf die Ansätze konzentrieren, die dich am meisten interessieren.

Ich muss sagen: Ich bin kein guter Scala Entwickler. Allerdings haben mir einige Konzepte sehr dabei geholfen bestimmte Dinge besser zu verstehen und auch meinen Python Code zu verbessern. Auch wenn man es nicht unbedingt anwendet, kann es durchaus bereichernd sein, sich mal nach links und rechts umzusehen und eventuell *versehentlich* etwas für sich mitzunehmen.


[//]: # (START CUSTOM SCRIPT)
[//]: # (START CONTENT LIST)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/sub-content/django-conceptual/de.md)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/sub-content/django-pure-impl/de.md)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/sub-content/cloud-conceptual/de.md)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/sub-content/django-and-aws-impl/de.md)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/sub-content/scala-akka-conceptual/de.md)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/sub-content/scala-akka-impl/de.md)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/sub-content/conclusion/de.md)
[//]: # (END CONTENT LIST)
[//]: # (END CUSTOM SCRIPT)
