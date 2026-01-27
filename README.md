# Monitoring
Système d'observabilité des services et d'infrastructure avec Grafana, prometheus, node-exporter et  opentelemetry

´Qui surveille le surveillant ?´

Observabilité = logs + métriques

# 1 Installation de Node exporter

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



  - https://github.com/Bhoopesh123/OpenTelemetry/blob/main/README.md
 
  - https://github.com/Bhoopesh123/OpenTelemetry/blob/main/README_OpenTelemetry_Metrics.md
