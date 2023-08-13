# Queueing und Web
Im folgenden werden drei Lösungsansätze für die selbe Aufgabenstellung vorgestellt. In diesen 
Ansätzen geht es darum, dass zwei Prozesse voneinander entkoppelt werden sollen:

1. eine Nutzeranfrage schreibt einen Eintrag in die Datenbank
2. mit dem neuen Eintrag wird ein "langsamer" Prozess getriggert

Die Antwort an den Nutzer oder die Nutzerin soll dabei aber möglichst sofort erfolgen und 
nicht extra auf die Verarbeitung des langsamen Prozesses gewartet werden.

## Über diesen Artikel
Dieser Artikel ist Teil von "JangasCodingplace" - meine persönliche Webseite. Mein Anliegen 
ist es nicht Tutorials zu schreiben - davon gibt es genug. Das können andere auch besser! 
Etwas was mir aber oft ein wenig fehlt: ganzheitliche, themenübergreifende Projekte und die 
Vermittlung von Konzepten. Darauf möchte ich mich ein wenig stürzen.

Zudem ist JangasCodingplace ein Ort, der mir dabei helfen soll, meine kleinen Freizeitprojekte 
abzuschließen und vorzustellen. Dieses Projekt ist ein Teil davon.

## Projektbeschreibung
[//]: # (START CUSTOM SCRIPT)
[//]: # (START MARKDOWNREF)
[//]: # (https://raw.githubusercontent.com/JangasCodingplace/jangas-codingplace-blogs/main/blogs/web-and-queues/assets/project-description/de.md)
[//]: # (END MARKDOWNREF)
[//]: # (END CUSTOM SCRIPT)


## Projektvorstellung
Dieses Projekt wird mehrfach mit verschiedenen Ansätzen gelöst. Konkret werden dabei folgende 
Lösungen vorgestellt:

- Python Django und der Python Queue
- Python Django, Amazon Simple Queue Service (SQS) und AWS Lambda
- Scala Play und Akka

Die Lösungen stehen jeweils für sich alleine und bestehen aus drei, bzw. vier Teilen:

1. Nochmalige Projektvorstellung
2. Schnellvorstellung des gewählten Ansatzes
3. Theoretische Einführung in die Lösung und Erklären der verwendeten Tools & Konzepte
4. Konkrete Implementierung der Lösung im vorgestellten Setup

Es ist nicht notwendig alles zu lesen. Nimm' dir die Teile heraus, die dich interessieren. 
Natürlich wird auch nicht jedes Tooling, Framework und Sprache von Grund auf erklärt. Es wird 
sich auf das beschränkt, was für das Projekt wichtig ist.


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
