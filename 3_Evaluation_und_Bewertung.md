# Evaluation und Bewertung

## Hochverfügbare Datenbank vs einzelne Instanz

Nachdem die Elixir-Anwendung hochverfügbar gemacht wurde, ist die Postgres-Datenbank der letzte verbleibende Single Point of Failure, innerhalb des Kubernetes-Clusters.

Um diesen zu Eliminieren muss die Datenbank hochverfügbar gemacht werden. Hierfür sind mindestens zwei PostgresDB Pods erforderlich, wobei einer als Primary und die restlichen als Replica arbeiten. Für das Load Balancing und Failover zwischen den DB Pods kommt pgpool-II zum Einsatz. Das Failover zwischen Primary und Replicas wird durch den postgresql-repmgr verwaltet. Dieser promotet eine Replica zum neuen Primary, falls der bestehende Primary ausfällt.

Das [HA Postgres Setup von Bitnami](https://github.com/bitnami/charts/tree/master/bitnami/postgresql-ha) besteht aus einer Sammlung von K8s Service Definitions, die über einen Helm Chart installiert werden. Helm ist im übertragenen Sinne ein Paketmanager für Kubernetes Service Definitions, die eine Anwendung deployen. Um Helm in MicroK8s zu nutzen, muss Helm erst als Addon aktiviert werden.

Nachdem die Parameter des [Bitnami Postgres HA Helm Charts](https://github.com/bitnami/charts/tree/master/bitnami/postgresql-ha#global-parameters) wie gewünscht konfiguriert wurden kann dieser via Helm installiert werden und ist danach einsatzbereit.

Im Vergleich zur Variante mit einer einzelnen Postgres Instanz ist dies ein signifikanter Mehraufwand. bei der Konfiguration und Deployment, da man zusätzlich zu den eigenen Service Definitions einen Helm Chart hat, dessen Konfiguration es auch entsprechend zu warten gilt. Bei der einzelnen Datenbank hingegen kann das Deployment einfach zusammen mit der Elixir-Anwendung mit dem selben Befehl erfolgen, da die Service Definitions im gleichen Verzeichnis liegen. Auch bei der Konfiguration war durchaus weniger zu beachten, da die Service Definitions für die einzelne PostgresDB mit kompose aus der docker-compose Datei generiert werden konnten.

Betrachtet man den Aspekt der Hochverfügbarkeit bzw. Ausfallsicherheit ist es durchaus sinnvoll auf einen hochverfügbaren Postgres Helm Chart zu setzen, da einzelne Nodes im Kubernetes Cluster ausfallen können und dies bei einer einzelnen Datenbank zu einem temporären Komplettausfall der Anwendung führen würde. Da im Rahmen des Praktikums jedoch nur ein Single Node Kubernetes Cluster genutzt wird, bietet der Einsatz des Postgres HA Charts keine Vorteile unter dem Aspekt der physischen Ausfallsicherheit. Der einzige nennenswerte Vorteil des HA DB Setups in Kombination mit einem Single Node K8s Cluster ist die unterbrechungsfreie Upgrademöglichkeit der Datenbank, jedoch ist diese Funktionalität für ein Test Deployment vernachlässigbar.

Daher bleiben wir für das Test Deployment auf einem Single Node K8s Cluster bei einer einzelnen Postgres-Instanz.

