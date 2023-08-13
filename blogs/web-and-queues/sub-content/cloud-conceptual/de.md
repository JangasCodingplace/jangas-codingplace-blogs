# Cloud (AWS)
In diesem Projekt wird AWS verwendet. AWS ist bekanntlich einer der drei großen Clound Anbieter (Amazon Web Services, Microsoft Azure, Google Cloud Console) - vermutlich ist AWS sogar der Größte der drei Anbieter.

Die angewendeten Services sind in ähnlicher Form aber in jedem der drei großen Anbieter verfügbar. Im folgenden wird sich also auf sehr generelle Aspekte beschränkt, die in diesem Projekt Anwendung finden:
- Serverless Functions
- Message Systems und Queues

## Serverless Functions
Serverless ist ein Cloud-Computing-Ausführungsmodell, bei dem der Cloud-Anbieter die Infrastruktur verwaltet, auf der Code ausgeführt wird. Man könnte sagen, dass der Name "serverless" ein wenig irreführend ist, da immer noch Server im Spiel sind. Der Unterschied besteht jedoch darin, dass Entwickler und Entwicklerinnen sich nicht um die Server-Verwaltung kümmern müssen - der Cloud Anbieter garantiert für Aktualität und Lauffähigkeit.

Serverless Functions sind eine spezielle Art von Serverless-Diensten, bei denen Entwickler Einzelfunktionen bereitstellen, die in Reaktion auf bestimmte Ereignisse (z. B. eine HTTP-Anfrage, eine neue File im Blob Storage, eine Änderung in einer Datenbank, uvm.) ausgeführt werden - das wird übrigens *ereignisgesteuerte Ausführung* genannt. Hierzu gibt es bei mir bereits [einen Artikel hier](https://jangascodingplace.com/blog/article/event-driven-programming-mit-serverless-functions).

In diesem Projekt werden die Lambda Funktionen von AWS genutzt.


## Queues und Message Systeme
In Microservice-Architekturen sind die Dienste oft so konzipiert, dass sie unabhängig und isoliert voneinander arbeiten. Um eine effiziente und entkoppelte Kommunikation zwischen diesen Diensten zu gewährleisten, werden häufig Warteschlangen (Queues) und Nachrichtensysteme (Message Systems) eingesetzt. 

Eine Queue ist die Implementierung einer Datenstruktur, die Nachrichten oder Aufgaben in der Reihenfolge ihrer Ankunft ablegt. In einer Microservice-Architektur kann ein Dienst eine Nachricht in eine Warteschlange senden, und ein anderer Dienst kann diese Nachricht später abholen und verarbeiten.

Dadurch wird eine Entkopplung ermöglicht. Dienste müssen nicht gleichzeitig online oder verfügbar sein. Auch Elastizität wird geboten. Wenn ein Dienst plötzlich eine große Anzahl von Anfragen erhält, können diese Anfragen in eine Warteschlange gestellt und schrittweise verarbeitet werden.

In diesem Projekt wird der Simple Queue Service von AWS genutzt.
