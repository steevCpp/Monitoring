# Monitoring
Système d'observabilité des services et d'infrastructure avec Grafana, prometheus, node-exporter et  opentelemetry

´Qui surveille le surveillant ?´

Observabilité = logs + métriques

# 1. Installation de Node exporter

Node Exporter est un agent à installer sur l’ensemble des machines à superviser, 
il permet de collecter périodiquement les métriques systèmes : Cpu, Ram, espace disque … 
Après installation ces métriques seront accessibles via une url du type http://:9100
Nos exporter collecte par default(CPU, RAM, DISQUE/FS).

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

Les metrics que l'on veut collecter sont: mountstats, logind, processes, ntp, systemd, tcpstat, wifi

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
Il collecte des métriques à partir de cibles configurées via une approche de "pull" HTTP. Son architecture est basée sur une base de données temporelle (TSDB) qui stocke les données avec une grande efficacité.
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

## 2.2.1 Création d'un service prometheus

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

## 2.2.2 Configuration de prometheus 
Toute la configuration de Prometheus est dans le fichier /etc/prometheus/prometheus.yml. 

## 2.2.2.1 Authentification
On sécurise l'accès à prometheus en mettant exigeant l'authentification pour les admins et pour les api.
On créé le fichier le web.yml, et on ajoute les informations **d'authentification d'api**, comme suite:
```
sudo vim /etc/prometheus/web.yml
```
On peut utilise le https://bcrypt-generator.com/ pour hasher le mot de passe.
```
basic_auth_users:
    admin: $2a$10$tMbmLWpgrFVDPXRBD7JlqOD/5.gIdWlZiBweoTtd5lVxa3ZMZrdcG
```
On vérifie la synthaxe du fichier web.yml

<img width="706" height="49" alt="image" src="https://github.com/user-attachments/assets/822ec031-dc28-4e32-9bb3-a03d82e17078" />

On rajoute l'option `--web.config.file=/etc/prometheus/web.yml` dans le fichier service prometheus.service

```
sudo vim /etc/systemd/system/multi-user.target.wants/prometheus.service
```
<img width="480" height="163" alt="image" src="https://github.com/user-attachments/assets/45abe9bf-623a-4f81-b73b-d62f8c373120" />

```
sudo systemctl daemon-reload
```

```
sudo systemctl restart prometheus
```
<img width="1213" height="485" alt="image" src="https://github.com/user-attachments/assets/936a760e-67fa-426c-99dc-ac03d6cefa32" />


On configure **l'authentification admin** prometheus pour ressoudre l'erreur 401


On rajoute le blocs suivant, en fin du fichier:

<img width="548" height="196" alt="image" src="https://github.com/user-attachments/assets/d3bfcbaa-c74e-47be-bc7f-a5f0531bbea1" />

On redemarre le service prometheus après modification.

```
sudo systemctl start prometheus
```
On peut voir apparaître nos modification sur le site prometheus, dans mon cas (http://localhost:9090/targets).
<img width="1348" height="442" alt="image" src="https://github.com/user-attachments/assets/b854803d-1e94-40d3-85a8-1ddba8f03f10" />



  - https://grafana.com/grafana/dashboards/1860-node-exporter-full/
  - https://prometheus.io/docs/guides/node-exporter/
  - https://github.com/Bhoopesh123/OpenTelemetry/blob/main/README.md
  - https://github.com/Bhoopesh123/OpenTelemetry/blob/main/README_OpenTelemetry_Metrics.md
