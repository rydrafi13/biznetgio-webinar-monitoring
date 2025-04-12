# biznetgio-webinar-monitoring

## Preparation Server
Neo Lite VPS


Order on <a href="https://portal.biznetgio.com/" target="_blank">Portal Biznetgio</a>.
```
OS: Ubuntu 22.04
Flavor: MS 4.2
CPU: 2 vCPU
RAM: 4 GB
Disk: 60 GB
```

Access to VM Ubuntu via ssh for linux os host
```
ssh -l rydrafi 103.x.x.x -i name-xxx.key
```

login as super user
```
sudo su
```

set hostname
```
hostnamectl set-hostname monitoring
```

set timezone
```
timedatectl set-timezone Asia/Jakarta
```

Update and Upgrade package
```
apt update -y && apt upgrade -y
```

## Install Prometheus Server
Create user and change ownership prometheus data directory
```
sudo useradd -M -U prometheus
mkdir -p  /var/lib/prometheus_data/
chown prometheus:prometheus -R /var/lib/prometheus_data/
```

Download Prometheus Server Binary File
```
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
```

Extract prometheus binary file
```
tar -zxvf prometheus-2.53.4.linux-amd64.tar.gz 
mv -v prometheus-2.53.4.linux-amd64 /opt/prometheus
chown prometheus:prometheus -R /opt/prometheus/
```

vim /etc/systemd/system/prometheus.service
```
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=on-failure
ExecStart=/opt/prometheus/prometheus \
  --config.file=/opt/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus_data/ \
  --storage.tsdb.retention.time=30d \
  --web.enable-remote-write-receiver

[Install]
WantedBy=multi-user.target
```

Reload daemon
```
sudo systemctl daemon-reload
```

Start and check service prometheus
```
sudo systemctl start prometheus.service
sudo systemctl enable prometheus.service
sudo systemctl status prometheus.service
sudo journalctl -u prometheus.service -f
```

Access prometheus dashboard
<img src="images/prometheus/1.prometheus-dashboard.png" alt="prometheus dashboard"/>

Check prometheus remote write receiver was enable
<img src="images/prometheus/2.promtheus-verify-remote-write-receiver.png" alt="prometheus remote write receiver"/>

## Config prometheus basic authentication
To generate password on here <a href="https://bcrypt.online/" target="_blank">Bcrypt Hash Generator</a>.

Create authentication file
```
vim /opt/prometheus/auth.yml
```

/opt/prometheus/auth.yml
```
#change password and generate with hash
basic_auth_users:
      admin: $2y$13$Xk98Up5g2MVy1w5mku9bx.ElrthyR58s9i7LKtgTzxkiizlgl2MmO 
```

Check validation auth file
```
/opt/prometheus/promtool check web-config /opt/prometheus/auth.yml
```

Enable Prometheus API basic authentication
```
vim /etc/systemd/system/prometheus.service
```

/etc/systemd/system/prometheus.service
```
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=on-failure
ExecStart=/opt/prometheus/prometheus \
  --config.file=/opt/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus_data/ \
  --storage.tsdb.retention.time=30d \
  --web.enable-remote-write-receiver \
  --web.config.file=/opt/prometheus/auth.yml #add

[Install]
WantedBy=multi-user.target
```

Reload daemon
```
sudo systemctl daemon-reload
```

Restart prometheus
```
systemctl restart prometheus.service
```

## Install Grafana Server
Install prequisite tools
```
sudo apt-get install -y apt-transport-https software-properties-common wget
```

Setup Grafana repository
```
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Install Grafana
```
sudo apt-get update
sudo apt-get install grafana
```

Start and Check Grafana Server
```
systemctl enable grafana-server --now
systemctl status grafana-server
```

Access grafana dashboard
<img src="images/grafana/1.grafana-login.jpeg" alt="grafana login"/>

Grafana dashboard
<img src="images/grafana/2.grafana-dashboard.png" alt="grafana dashboard"/>

Grafana add datasource connection
<img src="images/grafana/3.grafana-add-new-connection.png" alt="grafana add datasource"/>

Grafana config prometheus connection
<img src="images/grafana/4.grafana-config-prometheus.png" alt="grafana config prometheus"/>

Grafana config prometheus connection test and save
<img src="images/grafana/5.grafana-save-and-test.png" alt="grafana config prometheus test and save"/>
