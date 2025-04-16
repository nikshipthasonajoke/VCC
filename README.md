Ubuntu VirtualBox VM Monitoring with Prometheus and Grafana
Overview
This project demonstrates how to set up a virtual Ubuntu system using VirtualBox and monitor its performance using Prometheus and Grafana. It is intended for educational purposes related to understanding Linux system internals, virtualization, and basic observability tools.
Requirements
VirtualBox (6.x or later)
Ubuntu 20.04 or 22.04 ISO
Prometheus
Node Exporter
Grafana
Virtual Machine Setup
Open VirtualBox and click "New".
Use the following settings:
Name: Ubuntu_VM_A3
Type: Linux
Version: Ubuntu (64-bit)
RAM: 2048 MB
Processor: 2 CPUs
Video Memory: 64 MB
Graphics Controller: VBoxVGA
Create a 25 GB virtual hard disk (VDI).
Mount the Ubuntu ISO in the Optical Drive.
Boot and install Ubuntu normally.
Post-Installation Configuration
After Ubuntu installation, update the system:
sudo apt update && sudo apt upgrade -y
sudo apt install wget curl net-tools -y
Installing Prometheus
1. Create a Prometheus user
sudo useradd --no-create-home --shell /bin/false prometheus
2. Download and extract Prometheus
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-*.tar.gz
tar xvf prometheus-*.tar.gz
3. Move binaries and config files
sudo mv prometheus-* /etc/prometheus
sudo mv /etc/prometheus/prometheus /usr/local/bin/
4. Create the systemd service file
Create the file /etc/systemd/system/prometheus.service:
[Unit]
Description=Prometheus Monitoring
After=network.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries
Restart=always

[Install]
WantedBy=multi-user.target
5. Enable and start Prometheus
sudo systemctl daemon-reexec
sudo systemctl enable prometheus
sudo systemctl start prometheus
Access Prometheus at: http://localhost:9090
Installing Node Exporter
1. Download and run Node Exporter
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-*/
./node_exporter
2. Add Node Exporter to Prometheus config
Edit /etc/prometheus/prometheus.yml:
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
Restart Prometheus:
sudo systemctl restart prometheus
Installing Grafana
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana
Enable and start Grafana:
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
Access Grafana at: http://localhost:3000 Default login:
Username: admin
Password: admin
Folder Structure
project/
├── README.md
├── prometheus/
│   └── prometheus.yml
├── screenshots/
│   ├── vm-settings.png
│   ├── prometheus-ui.png
│   ├── grafana-dashboard.png
│   └── node-exporter.png
Troubleshooting
VirtualBox Guru Meditation
Set graphics controller to VBoxVGA
Increase video memory to at least 64 MB
Disable EFI in VM settings → System → Motherboard
Prometheus Target Not Detected
Ensure localhost:9100 is listed in prometheus.yml
Confirm Node Exporter is running on port 9100
Grafana Not Loading
Check Grafana status:
sudo systemctl status grafana-server
Confirm port 3000 is open and accessible
References
https://prometheus.io/docs/
https://grafana.com/docs/
https://github.com/prometheus/node_exporter
https://ubuntu.com/download
Author
Nikshiptha Sonajoke
B22CS050
