#!/bin/bash

# Update system
sudo apt update && sudo apt upgrade -y
sudo apt install wget curl net-tools software-properties-common -y

# Create Prometheus user
sudo useradd --no-create-home --shell /bin/false prometheus

# Download and install Prometheus
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-*.tar.gz
tar xvf prometheus-*.tar.gz
sudo mv prometheus-* /etc/prometheus
sudo mv /etc/prometheus/prometheus /usr/local/bin/

# Create Prometheus systemd service
cat <<EOF | sudo tee /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Monitoring
After=network.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus \\
  --config.file=/etc/prometheus/prometheus.yml \\
  --storage.tsdb.path=/var/lib/prometheus/ \\
  --web.console.templates=/etc/prometheus/consoles \\
  --web.console.libraries=/etc/prometheus/console_libraries
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Enable and start Prometheus
sudo systemctl daemon-reexec
sudo systemctl enable prometheus
sudo systemctl start prometheus

# Download and run Node Exporter
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-* && ./node_exporter &

# Install Grafana
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana -y

# Enable and started Grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

echo "Setup Complete!"
echo "Access Prometheus at http://localhost:9090"
echo "Access Grafana at http://localhost:3000 (login: admin/admin)"
