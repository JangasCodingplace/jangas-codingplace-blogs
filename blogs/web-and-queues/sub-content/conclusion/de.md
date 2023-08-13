# Zusammenfassung
Es wurden verschiedene Lösungen für das selbe Problem vorgestellt:
- Python Django pur mit der Python Queue
- Python Django & Amazon Simple Queue Service & Amazon Lambda
- Scala Play & Akka

Und natürlich haben alle Lösungen ihren Einsatzbereich.

Fangen wir mit dem offensichtlichen an: die Programmiersprache. Es macht natürlich wenig Sinn extra eine neue Programmiersprache für ein einzelnes Projekt zu lernen.

Sollte die Programmiersprache aber keine Rolle spielen, gibt es ein paar mehr Entscheidungskriterien.

Allgemein hat Django den Battery included Ansatz. Es bring wirklich extrem viel Funktionalität mit, die über das reine bereitstellen von Endpunkten hinaus geht. Dazu gehören insbesondere auch adminstrative Features oder auch Sachen wie ein Authentication System. Im Projekt habe ich mich davor etwas gedrückt. Nicht weil es komplex wäre, sondern einfach weil die Implementierung in Play im Vergleich recht umständlich geworden wäre. Je mehr Funktionalität und Developer Journey man haben möchte, desto besser fährt man vermutlich mit Django. Insbesondere wenn noch mit SQL Daten gearbeitet wird.

Die AWS Lösung wurde in diesem Beispiel jetzt nicht wirklich gebraucht. Eine solche Lösung fängt an Sinn zu machen, wenn man zwei getrennte Teams hat. Ein Team kümmert sich um den Web-Bereich und ein anderes Team um die Prompts und die Aufarbeitung der Rückgaben. Hier hat man dann eine Super Entkopplung der Dienste. Insbesondere wenn die Anforderungen an den LLM Service wachsen.
Aber auch das kann allgemein Geschmacksache sein. Es gibt Teams, die funktionieren besser in einer monolithischen Codebasis und es gibt Teams die funktionieren besser in Microservices. Ich habe beides erlebt. Und für mich persönlich muss ich mittlerweile sagen, dass ich eine einzelne Codebasis für kleine bis mittelgroße Projekte schöner finde, als viele kleine.

Zuletzt natürlich noch die Art und weise der Nebenläufigkeit. Es stellt sich so ein wenig die Frage, ob Django und Queues oder Scala mit seinen Actors da die Nase vorn hat. Hier würde ich sagen: weder, noch. Die Queue Implementierung von Django ist mit Sicherheit nicht die effizienteste Lösung. Wenn wir noch etwas mehr Tooling drauf werfen, kann mit Celery eine schöne und stabile Lösung entstehen. Scala Play & Akka wiederum bleiben in ihrem Setting. Das Scala Setup kann auf einem Server stattfinden, oder über mehrere verteilt werden, ohne dass es abseits von Akka wirklich mehr Tooling bedarf um ggf. den State der Anwendung zu verteilen.
Aber auch hier um fair zu sein: das braucht man bei einer Anwendung wie dieser nicht wirklich. Es ist schön für den Stammtisch in der Kneipe, für mehr aber auch nicht. Hier würde man mit hoher Wahrscheinlichkeit die falsche Stelle optimieren.

Ich komme also nicht umher zu sagen, dass meiner Meinung nach die beste Lösung für das Problem die reine Django Implementierung ist. Sie funktioniert einfach, ist extrem gut zu erweitern und macht das Szenario nicht komplizierter, als es sein müsste.
