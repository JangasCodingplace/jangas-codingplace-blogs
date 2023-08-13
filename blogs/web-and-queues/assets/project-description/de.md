# Projektbeschreibung
Dieses Projekt stellt eine Teilfunktion einer meiner Anwendungen "Dev-Guro" dar. 
In dieser Teilfunktion werden Dev-Logs gespeichert. Sollten Tech-Stack spezifische 
Dinge im Log erwähnt werden, so sollen diese in Form von Tags aus den Logs gezogen 
und gespeichert werden.

Hier mal ein konkretes Beispiel eines Logs:
> Heute habe ich eine Anwendung mit Django geschrieben. Diese Anwendung schreibt
> Informationen nach Elasticsearch und ist auf ImaginaryNoneExistence Cloud deployed.

folgende Informationen sollen extrahiert werden:
- Django
- Elasticsearch
- ImaginaryNoneExistence Cloud

Der Extraktionsprozess soll im Hintergrund passieren. Für den Nutzer oder die 
Nutzerin die einen Log schreibt ist es nicht wichtig zu sehen, welcher Tech-Stack 
aus dem Log gezogen wird. Es ist nur wichtig, möglichst schnell zu antworten.
