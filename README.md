# Monitoring
Système d'observabilité des services et d'infrastructure avec Grafana, prometheus, node-exporter et  opentelemetry

´Qui surveille le surveillant ?´

Observabilité = logs + métriques

# 1. Installation de Node exporter

Node Exporter est un agent à installer sur l’ensemble des machines à superviser, 
il permet de collecter périodiquement les métriques systèmes : Cpu, Ram, espace disque … 
Après installation ces métriques seront accessibles via une url du type http://:9100
## 1.1 Récuperation et extraction du binaire
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
```

```
tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz
```

```
cd node_exporter-1.10.2.linux-amd64 && sudo mv node_exporter /usr/local/bin/
```
## 1.2 Création de l'utilisateur service et le service systemd

utilisateur service sans répertoire et sans shell

```
sudo useradd -rs /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
Création d'un service node-exporter

```
sudo vim /lib/systemd/system/node_exporter.service
```
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter \
   --collector.mountstats \
   --collector.logind \
   --collector.processes \
   --collector.ntp \
   --collector.systemd \
   --collector.tcpstat \
   --collector.wifi
Restart=always

[Install]
WantedBy=multi-user.target
```

Active le service 
```
sudo systemctl enable node_exporter.service
```

Démarre le service
```
sudo systemctl daemon-reload && sudo systemctl start node_exporter.service
```

```
journalctl -u node_exporter.service -f
```
<img width="1087" height="379" alt="image" src="https://github.com/user-attachments/assets/60cf378f-4e5b-461f-bb44-1f59f357ea5c" />

# 2. Installation de Prometheus
Prmotheus est générateur d'alerte, une base de données de séries temporelles qui enregistre des métriques en temps réel, avec une capacité d'acquisition élevée.
Ces métriques peuvent ensuite être interrogé à l'aide d'un langage de requête simple (PromQL) et peuvent également servir à générer des alertes   
une url du type http://:9090

## 2.1 Récuperation et extraction du binaire

```
 wget https://github.com/prometheus/prometheus/releases/download/v3.5.1/prometheus-3.5.1.linux-amd64.tar.gz

tar xvzf prometheus-3.5.1.linux-amd64.tar.gz
```
<img width="488" height="49" alt="image" src="https://github.com/user-attachments/assets/0b5119a9-65da-4765-8d58-96d8691afdaf" />

- prometheus.yml : Fichier de configuration
- prometheus : Binaire exécutable
- promtool : Outils de vérification de syntaxe, de configuration

## 2.2 Création d'un service prometheus

L'utilisateur service prometheus 
```
sudo useradd -rs /bin/false prometheus
```

```
sudo mv prometheus /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/prometheus
```

```
sudo mv promtool /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/promtool
```

```
sudo mkdir /etc/prometheus

sudo cp prometheus.yml /etc/prometheus/
```

```
sudo mkdir -p /data/prometheus

sudo chown -R prometheus:prometheus /data/prometheus /etc/prometheus/*
```

```
sudo vim /lib/systemd/system/prometheus.service
```

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path="/data/prometheus" \
#  --web.console.templates=/etc/prometheus/consoles \
#  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-admin-api

Restart=always

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl enable prometheus

sudo systemctl start prometheus

sudo systemctl daemon-reload
```
<img width="992" height="406" alt="image" src="https://github.com/user-attachments/assets/8aa6ccc2-7196-4e63-bbd4-c7d9fe46663c" />




  - https://prometheus.io/docs/guides/node-exporter/
  - https://github.com/Bhoopesh123/OpenTelemetry/blob/main/README.md
  - https://github.com/Bhoopesh123/OpenTelemetry/blob/main/README_OpenTelemetry_Metrics.md
