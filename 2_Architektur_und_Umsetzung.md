# Architektur und Umsetzung

## DoIt-Anwendung

Unsere Anwendung trägt den Namen „DoIt“. Mit dieser können ToDo-Listen erstellt und verwaltet werden. Die Anwendung wurde in Elixir mit dem Phoenix-Framework entwickelt. Für das Frontend kam zusätzlich das CSS-Framework "Tailwind CSS" zum Einsatz.

### Registrierung und Anmeldung

Um DoIt zu benutzen, muss zunächst ein Nutzerkonto erstellt werden. Bei der Registrierung müssen die Felder E-Mail (String) und Passwort (String) ausgefüllt werden. Bei der E-Mail-Adresse wird zusätzlich geprüft, ob der String keine Leerzeichen und nur ein „@“-Zeichen enthält. Die E-Mail-Adresse kann maximal 160 Zeichen lang sein. Bei der Anmeldung muss der Nutzer sich mit seinen zuvor eingegebenen Daten authentifizieren. Falls die eingegebene Kombination falsch ist, wird eine entsprechende Fehlermeldung angezeigt und die Anmeldedaten müssen erneut eingegeben werden. Bei einer erfolgreichen Registrierung oder Anmeldung wird dem Nutzer die Liste der von ihm erstellten Todos angezeigt.

### Todos Verwaltung

Ein Todo besteht aus dem Titel (String), der Beschreibung (String), der Wichtigkeit (Integer, default: 0) und einem „Done“ (Boolean), das signalisiert, ob die Aufgabe erledigt wurde. Jedes Todo wird einem einzigen Nutzer zugewiesen. Durch das Klicken auf „Add Todo“ und unter Angabe eines Todo-Textes wird ein neues Todo erstellt. Dieses wird dann dem Nutzer zugewiesen und in seiner Übersicht angezeigt. Der Nutzer ist in der Lage seine Todos zu bearbeiten. Damit ein Todo bearbeitet werden kann, muss der Nutzer auf „Edit“ klicken. Der Nutzer wird dann auf eine Bearbeitungsseite weitergeleitet, wo er die Werte aller Felder ändern kann. Nach dem Eintragen der gewünschten Werte, muss „Save Todo“ geklickt werden und dem Nutzer wird die Liste aller Todos angezeigt.
Um ein Todo zu löschen, muss auf „Delete“ geklickt werden. Danach wird das Todo gelöscht und der Nutzer auf die Todo-Listen Übersicht weitergeleitet. 

Wie schon in der Einleitung erwähnt, nutzt die Anwendung den im Phoenix-Framework standardmäßig integrierten PubSub-Kanal welcher das Publish-Subscribe Nachrichtenübermittlungsmuster implementiert. Dadurch kann sichergestellt werden, dass sich alle Browserinstanzen selbstständig asynchron über die Websocketverbindung synchronisieren können. In unserer Anwendung funktioniert diese Synchronisation sogar über mehrere Knoten eines Clusters hinweg. Alle möglichen Änderungen an den ToDo´s werden dabei via Broadcast-Nachricht zum Thema "todos" angezeigt. Alle laufenden Browserinstanzen, die sogenannten "subscriber" zum Thema "todos", werden dann über die ausgeführten Änderungen informiert und können sich darauf basierend asynchron aktualisieren.

### Kontoverwaltung

Ein Nutzer-Eintrag besteht aus der E-Mail Adresse, Passwort, verschlüsseltem Passwort und E-Mail-Bestätigungsdatum. Jedem Nutzer wird eine Liste von Todos zugewiesen. Der Nutzer kann seine Personaldaten durch das Klicken auf den „Settings“ Knopf bearbeiten. Er wird dann zu einer Seite weitergeleitet, wo seine E-Mail Adresse und sein Passwort geändert werden können. Um die E-Mail Adresse zu ändern, müssen zwei Felder ausgefüllt werden: Die neue E-Mail Adresse und das aktuelle Passwort. Die Änderung muss dann durch das Klicken auf den „Change email“ Knopf bestätigt werden. Die Änderung von dem Passwort erfolgt in ähnlicher Weise, die folgenden drei Felder müssen hierfür ausgefüllt werden: Zweifach das neue Passwort (ein weiteres Mal zur Bestätigung) und das aktuelle Passwort. Die Passwortänderung muss durch das Klicken auf den „Change password“ Knopf bestätigt werden.

![doit-app](https://github.com/Elixir2K8s/docs/blob/main/doit.PNG)

## Docker-Compose

### Docker-Compose

Bei Multi-Container Anwendungen sind Tools wie Docker-Compose hilfreich um die Orchestrierung zu definieren. In der [docker-compose.yml]( https://github.com/Elixir2K8s/doit/blob/master/docker-compose.yml) YAML-Datei befindet sich die Docker-Compose Konfiguration unserer DoIt Anwendung, die Build- und Ausführungsprozesse definiert. Unter Verwendung von Docker-Compose müssen die Container Konfigurationen nicht im Rahmen mehrerer vielzeiliger Befehle konfiguriert werden, sondern die gesamte Konfiguration wird in der Docker-Compose YAML gespeichert. Die durch Docker-Compose konfigurierten Container können im Gegensatz zum normalen `docker run` Befehl gleichzeitig via `docker-compose up` gestartet werden. Ein weiterer Vorteil von Docker-Compose ist, dass die Abhängigkeiten zwischen Containern konfiguriert werden können.
Docker-Compose wird zusammen mit Docker mit Hilfe von `snap install docker` installiert.

### Testdeployment der Anwendung

Um die Anwendung zu deployen, sollten zuerst die Images mit `docker-compose build`, ein Wrapper der für jeden konfigurierten Container `docker build` ausführt, gebaut werden. Anschließend können die Container mit `docker-compose up` erstellt und ausgeführt werden.
Zuletzt muss das PostgresSQL Datenbankschema über den Elixir Container erstellt werden. Dazu wird `docker-compose run elixir /app/bin/doit eval "Doit.Release.create"` verwendet.
Dann muss das Datenbankschema mit `docker-compose run elixir /app/bin/doit eval "Doit.Release.migrate"` migriert werden.

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

Um das Container Image zu bauen und im Anschluss auf die Image Registry zu pushen, muss das [Repo mit der ToDo-Anwendung](https://github.com/Elixir2K8s/doit) geklont werden, falls dies nicht schon erfolgt ist. Im `doit` Verzeichnis wird nun der Befehl `docker-compose build` ausgeführt um die Anwendung zu bauen. Mit dem Kommando `docker-compose push` wird das Image der Anwendung auf die Registry befördert.

### Erstellung der Service Definitions

Im Gegensatz zu Docker-Compose benötigt bei Kubernetes jeder Service mehrere Dateien die diesen definieren, auch Service Definitions genannt. Die Minimalkonfiguration eines Dienstes besteht hierbei aus Deployment und Service. Hierbei konfiguriert das Deployment den Dienst, bspw. wie viele Instanzen von dem Container laufen sollen, welches Container Image genutzt werden soll und welche Environment Variablen, während der Service für die Veröffentlichung zuständig ist, also unter welchem Hostname, Port, etc. der Dienst erreichbar sein soll.

Die Erstellung der Service Definitions von Hand ist ein sehr zeitintensiver Prozess, weshalb wir das Tool [kompose](https://kompose.io/) nutzen. um aus der [docker-compose.yaml](https://github.com/Elixir2K8s/doit/blob/master/docker-compose.yml) der ToDo-Anwendung die grundlegenden Service Definitions zu generieren. Da die autogenerierten Service Definitions jedoch nicht perfekt waren mussten noch einige kleine Änderungen, bspw. bei den Ports oder der Anzahl der Replicas vorgenommen werden. Auch der Headless Service für die AutoDiscovery der Elixir-Pods untereinander sowie Umgebungsvariablen, welche für die Anwendungskonfiguration notwendig sind, musste noch manuell ergänzt werden. Gleiches gilt auch für den HTTP Ingress Controller, der die Elixir-Anwendung via HTTPS auf Port 443 zugänglich macht.

### Deployment der Anwendung

Um die Anwendung auf dem MicroK8s Kubernetes Cluster zu deployen wird jetzt innerhalb vom [k8s-service-definitions](https://github.com/Elixir2K8s/k8s-service-definitions) Directory das kubectl Utility verwendet. Im ersten Schritt werden mit `microk8s.kubectl apply -f` die Service Definitionen aus dem aktuellen Ordner auf das Kubernetes Cluster angewendet. Im nächsten Schritt muss mit `microk8s.kubectl exec deployment.apps/elixir /app/bin/doit eval "Doit.Release.create"` die Datenbank erzeugt werden. Zuletzt wird mit `kubectl exec deployment.apps/elixir /app/bin/doit eval "Doit.Release.migrate"` die Datenbankmigration ausgeführt.

Jetzt sollte die Anwendung unter https://localhost erreichbar sein.

## Grafische Darstellung der Kubernetes Architektur
![application-diagram](https://github.com/Elixir2K8s/docs/blob/main/application_diagram.png)

