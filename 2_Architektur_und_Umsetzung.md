# Architektur und Umsetzung

## DoIt-Anwendung

Unsere Anwendung trägt den Name „DoIt“. Mit deren Hilfe kann man die ToDo-Listen erstellen und verwalten. Die Anwendung wurde in Elixir geschrieben, mit Hilfe von HTML und CSS Tailwind.

### Registrierung und Anmeldung

Um DoIt zu benutzen, muss man ein Nutzerkonto erstellen. Bei der Registrierung müssen E-Mail (String) und Passwort (String) Felder ausgefüllt werden. E-Mail-Adresse wird zusätzlich geprüft, ob das String keine Leerzeichen und genau ein „@“ enthält. E-Mail-Adresse kann maximal einer Länge von maximal 160 Zeichen sein. Bei der Anmeldung wird der Nutzer nach seiner E-Mail-Adresse und Passwort gefragt. Falls die eingegebene Kombination falsch ist, wird eine entsprechende Fehlernachricht angezeigt und die Anmeldungsdateien erneut eingegeben werden müssen. Im Fall der erfolgreichen Registrierung oder Anmeldung wird dem Nutzer die Liste von ihm erstellten Todos angezeigt.

### Todos Verwaltung

Ein Todo besteht aus dem Titel (String), der Beschreibung (String), der Wichtigkeit (Integer, default: 0) und einem „Done“ Boolean, das bezeichnet, ob die Aufgabe erledigt ist. Jedes Todo wird auch einem einzigen Nutzer zugewiesen. Durch das Klicken auf „New Todo“ wird der Nutzer auf eine Seite weitergeleitet, wo ein neues Todo erstellt werden kann. Nach dem Ausfüllen von oben genannten Felder, kann man auf „Add Todo“ klicken, um es zu speichern. Das Todo wird dann dem aktuell eingeloggten Nutzer zugewiesen und in seiner Liste angezeigt.
Der eingeloggte Nutzer ist in der Lage seine Todos zu bearbeiten. Damit ein Todo bearbeitet werden kann, muss der Nutzer auf „Edit“ klicken. Der Nutzer wird dann auf eine Bearbeitungsseite weitergeleitet, wo er die Werte allen Felder ändern kann. Nach dem Eintragen von gewünschten Werten, muss „Save Todo“ geklickt werden und dem Nutzer wird die Liste allen Todos angezeigt.
Um ein Todo zu löschen, muss „Delete“ geklickt werden. Danach wird ein Todo gelöscht und der Nutzer auf die Todo-Liste weitergeleitet. 

### Kontoverwaltung

Ein Nutzer besteht aus der E-Mail Adresse, Passwort, verschlüsseltes Passwort und Bestätigungsdatum. Jedem Nutzer wird auch eine Liste von Todos zugewiesen. Der Nutzer kann seine Personaldaten durch das Klicken auf den „Settings“ Knopf bearbeiten. Er wird dann zur Seite weitergeleitet, wo seine E-Mail Adresse und Passwort geändert werden können. Um die E-Mail Adresse zu ändern, müssen zwei Felder ausgefüllt werden: die neue E-Mail Adresse und das aktuelle Passwort. Die Änderung muss dann durch das Klicken auf den „Change email“ Knopf bestätigt werden. Die Änderung von Passwort erfolgt in ähnlicher Weise: die drei Felder müssen ausgefüllt werden: das Neue Passwort wiederholt in zwei verschiedenen Felder und das aktuelle Passwort. Die Passwortänderung muss durch das Klicken auf den „Change password“ Knopf bestätigt werden.

## Docker Compose

### Docker-Compose

Bei Multi-Container Anwendungen sind die Tools hilfreich, die diese Anwendungen definieren. Zu diesem Zweck wird in unserem Projekt Docker-Compose verwendet. In [docker-compose.yml]( https://github.com/Elixir2K8s/doit/blob/master/docker-compose.yml) YAML Datei befindet sich die Definition unserer DoIt Anwendung, damit wird es gewährleistet, dass die Anwendung sich bei jedem Nutzer gleich verhaltet. Unter Verwendung von Docker-Compose muss die Docker Konfiguration nicht im Rahmen von mehreren vielzeilige Befehle konfiguriert werden, sondern wird die ganze Konfiguration in dieser YAML Datei eingeschlossen. Die durch Docker-Compose konfigurierte Containers können auch im Gegensatz zum normalen `docker run` Befehl gleichzeitig gestartet werden. Ein anderer Vorteil von Docker-Compose ist, dass die Abhängigkeiten zwischen Containers in YAML Datei konfiguriert werden können.
Docker-Compose kann mit Hilfe von `snap install docker` installiert werden

### Testdeployment der Anwendung

Um die Anwendung zu deployen, sollte zuerst das Image mit `docker-compose build`, ein Wrapper, das für jeden Container `docker build` läuft, gebaut werden, dann können die Containers mit `docker-compose up` erstellt und gestartet werden.
Jetzt muss die erste Datenbank mit Elixir Pod erstellt werden. Dazu wird `docker-compose run elixir /app/bin/doit eval "Doit.Release.create"` verwendet.
Dann sollte sie mit `docker-compose run elixir /app/bin/doit eval "Doit.Release.migrate"` migriert werden.

Jetzt sollte die Anwendung unter http://localhost:4000 erreichbar sein.


## Kubernetes

### Auswahl der Kubernetes Distribution

Ähnlich wie bei Linux gibt es nicht nur ein Kubernetes, sondern viele verschiedene Distributionen, z. B. Minikube, MicroK8s, K3s, OpenShift und Rancher nur um einige davon zu nennen. Analog zu den Linux Distributionen, bietet jede Kubernetes (K8s) Distribution verschiedene Featuresets. 

Unser Ziel war es eine leichtgewichtige und einfach zu konfigurierende K8s Distribution zu nutzen, wobei sich aufgrund des verwendeten Host-OSes Ubuntu, MicroK8s anbot. MicroK8s wird per snap installiert und ist dadurch vom restlichen Betriebsystem weitgehend abgekapselt. Dies wäre bei alternativen K8s Distros wie Minikube und K3s nicht der Fall, da diese nativ auf dem Host installiert werden.

### Installation und Konfiguration der Kubernetes-Distribution MicroK8s

Die Installation von MicroK8s erfolgt via CLI mit dem Befehl `snap install microk8s`. Im Anschluss kann das lokale Kubernetes Cluster mit `microk8s.start` bzw. `microk8s.stop` gestartet oder gestoppt werden.

Damit die ToDo-Anwendung auf dem lokalen Cluster lauffähig ist müssen noch einige Kubernetes Komponenten aktiviert werden. Zum einen muss der lokale Storage Provider mit `microk8s enable storage` aktiviert werden, damit die Postgres-Datenbank(en) ihre Daten persistent speichern können und nach einem Redeployment bzw. Container-Restart ihre Daten nicht verlieren.

Weiterhin muss noch der lokale DNS-Provider mit `microk8s enable dns` aktiviert werden, damit die Elixir-Pods auf die Postgres-DB zugreifen können bzw. ihre Replicas via DNS Discovery finden können.

Zuletzt wird noch das Feature Ingress Controller mit `microk8s enable ingress` aktiviert, um einen sicheren Zugriff via HTTPS auf die ToDo-Anwendung zu ermöglichen.

### Bereitstellung der Container Images

Damit die Anwendung auf Kubernetes deployed werden kann müssen die Container Images auf einer Image Registry verfügbar sein. Für Docker Images ist dies standardmäßig [Docker Hub](https://hub.docker.com/). Für Entwicklungszwecke und private Anwendungen bietet es sich hingegen an eine eigene Image Registry zu betreiben. Hierfür nutzen wir die offizielle [Docker Registry 2.0 Implementierung](https://hub.docker.com/_/registry). Diese ist für Testzwecke ausreichend, jedoch sollte von einem Produktiveinsatz abgesehen werden, da Features wie beispielsweise eine Garbage-Collection fehlen. Hierfür sollte man auf umfangreichere Lösungen wie die [Harbor Registry](https://goharbor.io/) setzen. Für unseren Einsatzzweck, das pushen und pullen eines einzigen Images, ist die Docker Registry 2.0 jedoch völlig ausreichend.

Die Docker Registry wurde der Einfachheit halber mit einer docker-compose.yaml konfiguriert und kann nach dem Clonen des [image-registry repos](https://github.com/Elixir2K8s/image-registry) einfach via `docker-compose up -d` gestartet werden. Nun ist die Image Registry einsatzbereit.

Um das Container Image zu bauen und im Anschluss auf die Image Registry zu pushen, muss das [Repo mit der ToDo-Anwendung](https://github.com/Elixir2K8s/doit) geclont werden, falls dies nicht schon erfolgt ist. Im `doit` Verzeichnis wird nun der Befehl `docker-compose build` ausgeführt um die Anwendung zu bauen. Mit dem Kommando `docker-compose push` wird das Image der Anwendung auf die Registry befördert.

### Erstellung der Service Definitions

Im Gegensatz zu Docker-Compose benötigt bei Kubernetes jeder Service mehrere Dateien die diesen definieren, auch Service Definitions genannt. Die Minimalkonfiguration eines Dienstes besteht hierbei aus Deployment und Service. Hierbei konfiguriert das Deployment den Dienst, bspw. wie viele Instanzen von dem Container laufen sollen, welches Container Image genutzt werden soll und welche Environment Variablen, während der Service für die Veröffentlichung zuständig ist, also unter welchem Hostname, Port, etc. der Dienst erreichbar sein soll.

Die Erstellung der Service Definitions von Hand ist ein sehr zeitintensiver Prozess, weshalb wir das Tool [kompose](https://kompose.io/) nutzen. um aus der [docker-compose.yaml](https://github.com/Elixir2K8s/doit/blob/master/docker-compose.yml) der ToDo-Anwendung die grundlegenden Service Definitions zu generieren. Da die autogenerierten Service Definitions jedoch nicht perfekt waren mussten noch einige kleine Änderungen, bspw. bei den Ports oder der Anzahl der Replicas vorgenommen werden. Auch der Headless Service für die AutoDiscovery der Elixir-Pods untereinander musste noch manuell ergänzt werden. Gleiches gilt auch für den HTTP Ingress Controller, der die Elixir-Anwendung via HTTPS auf Port 443 zugänglich macht.

### Deployment der Anwendung

Um die Anwendung auf dem MicroK8s Kubernetes Cluster zu deployen wird jetzt innerhalb vom [k8s-service-definitions](https://github.com/Elixir2K8s/k8s-service-definitions) Directory das kubectl Utility verwendet. Im ersten Schritt werden mit `microk8s.kubectl apply -f` die Service Definitionen aus dem aktuellen Ordner auf das Kubernetes Cluster angewendet. Im nächsten Schritt muss mit `microk8s.kubectl exec deployment.apps/elixir /app/bin/doit eval "Doit.Release.create"` die Datenbank erzeugt werden. Zuletzt wird mit `kubectl exec deployment.apps/elixir /app/bin/doit eval "Doit.Release.migrate"` die Datenbankmigration ausgeführt.

Jetzt sollte die Anwendung unter https://localhost erreichbar sein.

