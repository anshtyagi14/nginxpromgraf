# Installing and setting up Nginx Prometheus Exporter, Prometheus, and Grafana for system log visualization

This guide will help you install and set up Nginx Prometheus Exporter, Prometheus, and Grafana to visualize Nginx logs on Grafana.

## Requirements

Linux-based operating system (Ubuntu, Debian, etc.)

## Initial setup

### Step 1: Update your system

```console
$ sudo apt update
$ sudo apt upgrade
```

### Step 2: Install and configure Nginx

```console
$ sudo apt install nginx
$ sudo vim /etc/nginx/nginx.conf
```

Add the following configuration inside the http block

```yml
server {
    listen 127.0.0.1:8080;
    server_name localhost;

    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}
```

Restart Nginx

```console
$ sudo systemctl restart nginx
```

### Step 3: Install Nginx Prometheus Exporter

Download and extract Nginx Prometheus Exporter

```console
$ wget https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v0.11.0/nginx-prometheus-exporter_0.11.0_linux_amd64.tar.gz
$ tar xvzf nginx-prometheus-exporter_0.11.0_linux_amd64.tar.gz
```

Copy the binary to /usr/local/bin/ and set permissions

```console
$ sudo cp nginx-prometheus-exporter /usr/local/bin/
$ sudo useradd --no-create-home --shell /bin/false nginx-prometheus-exporter
$ sudo chown nginx-prometheus-exporter:nginx-prometheus-exporter /usr/local/bin/nginx-prometheus-exporter
```

Create a new systemd service file for Nginx Prometheus Exporter

```console
$ sudo vim /etc/systemd/system/nginx-prometheus-exporter.service
```

Paste the following content into the file

```yml
[Unit]
Description=Nginx Prometheus Exporter
After=network.target
Wants=network.target

[Service]
Type=simple
User=nginx-prometheus-exporter
Group=nginx-prometheus-exporter
ExecStart=/usr/local/bin/nginx-prometheus-exporter -nginx.scrape-uri http://127.0.0.1:8080/nginx_status

[Install]
WantedBy=multi-user.target
```

Reload systemd, enable, and start Nginx Prometheus Exporter

```console
$ sudo systemctl daemon-reload
$ sudo systemctl enable nginx-prometheus-exporter
$ sudo systemctl start nginx-prometheus-exporter
$ systemctl status nginx-prometheus-exporter
```

### Step 4: Install Prometheus

Download and extract Prometheus

```console
$ wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
$ tar xvzf prometheus-2.43.0.linux-amd64.tar.gz
```

Create necessary directories and copy files

```console
$ sudo mkdir /var/lib/prometheus
$ sudo mkdir /etc/prometheus
$ sudo mkdir /etc/prometheus/consoles
$ sudo mkdir /etc/prometheus/console_libraries

$ sudo cp prometheus-2.43.0.linux-amd64/prometheus /usr/local/bin/
$ sudo cp prometheus-2.43.0.linux-amd64/promtool /usr/local/bin/
$ sudo cp prometheus-2.43.0.linux-amd64/prometheus.yml /etc/prometheus/
$ sudo cp -r prometheus-2.43.0.linux-amd64/consoles /etc/prometheus
$ sudo cp -r prometheus-2.43.0.linux-amd64/console_libraries/ /etc/prometheus
```

Create a Prometheus user and set permissions

```console
$ sudo useradd --no-create-home --shell /bin/false prometheus

$ sudo chown prometheus:prometheus /var/lib/prometheus
$ sudo chown prometheus:prometheus /etc/prometheus
$ sudo chown -R prometheus:prometheus /etc/prometheus/consoles
$ sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

Modify the Prometheus configuration file

```console
$ sudo vim /etc/prometheus/prometheus.yml
```

Add the following content under the scrape_configs section

```yml
  - job_name: "nginx"
    static_configs:
      - targets: ["localhost:9113"]
```

Create a new systemd service file for Prometheus

```console
$ sudo vim /etc/systemd/system/prometheus.service
```

Paste the following content into the file

```yml
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Reload systemd, enable, and start Prometheus

```console
$ sudo systemctl daemon-reload
$ sudo systemctl enable prometheus
$ sudo systemctl start prometheus
$ systemctl status prometheus
```

### Step 5: Install Grafana

Install Grafana dependencies

```console
$ sudo apt-get install -y adduser libfontconfig1
```

Download and install Grafana

```console
$ wget https://dl.grafana.com/enterprise/release/grafana-enterprise_9.4.7_amd64.deb
$ sudo dpkg -i grafana-enterprise_9.4.7_amd64.deb
```

Enable and start Grafana

```console
$ sudo systemctl enable grafana-server
$ sudo systemctl start grafana-server
$ systemctl status grafana-server
```

### Step 6: Configure Grafana to visualize system logs

1. Open Grafana in your browser at http://your-server-ip:3000 and log in with the default credentials (username: admin, password: admin).
2. Click on the Configuration (gear icon) in the left sidebar, then Data sources, and click Add Data source.
3. Select Prometheus as the data source.
4. Under HTTP, set the URL to http://localhost:9090.
5. Click Save & test to save the configuration and test the connection.
6. To import a pre-built dashboard, click on Dashboards (four squares icon) in the left sidebar, then click New, and select Import.
7. Click on Import dashboard and enter _ _ _ _ _  in the Import via grafana.com field, then click Load.
8. Select Prometheus as the data source from the dropdown and click Import.

Now you have successfully installed Node Exporter, Prometheus, and Grafana and visualized system logs on Grafana using a prebuilt dashboard.


