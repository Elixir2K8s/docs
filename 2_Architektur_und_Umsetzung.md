# Architektur und Umsetzung

## Kubernetes

### Auswahl der Kubernetes Distribution

Ähnlich wie bei Linux gibt es nicht nur ein Kubernetes, sondern viele verschiedene Distributionen, z. B. Minikube, MicroK8s, K3s, OpenShift und Rancher nur um einige davon zu nennen. Analog zu den Linux Distributionen, bietet jede Kubernetes (K8s) Distribution verschiedene Featuresets. 

Unser Ziel war es eine leichtgewichtige und einfach zu konfigurierende K8s Distribution zu nutzen, wobei sich aufgrund des verwendeten Host-OSes Ubuntu, MicroK8s anbot. MicroK8s wird per snap installiert und ist dadurch vom restlichen Betriebsystem weitgehend abgekapselt. Dies wäre bei alternativen K8s Distros wie Minikube und K3s nicht der Fall, da diese nativ auf dem Host installiert werden.

### Installation und Konfiguration der K8s Distribution MicroK8s

Die Installation von MicroK8s erfolgt via CLI mit dem Befehl `snap install microk8s`. Im Anschluss kann das lokale Kubernetes Cluster mit `microk8s.start` bzw. `microk8s.stop` gestartet oder gestoppt werden.

Damit die ToDo-Anwendung auf dem lokalen Cluster lauffähig ist müssen noch einige Kubernetes Komponenten aktiviert werden. Zum einen muss der lokale Storage Provider mit `microk8s enable storage` aktiviert werden, damit die Postgres-Datenbank(en) ihre Daten persistent speichern können und nach einem Redeployment bzw. Container-Restart ihre Daten nicht verlieren.

Weiterhin muss noch der lokale DNS-Provider mit `microk8s enable dns` aktiviert werden, damit die Elixir-Pods auf die Postgres-DB zugreifen können bzw. ihre Replicas via DNS Discovery finden können.

Zuletzt wird noch das Feature Ingress Controller mit `microk8s enable ingress` aktiviert, um einen sicheren Zugriff via HTTPS auf die ToDo-Anwendung zu ermöglichen.

