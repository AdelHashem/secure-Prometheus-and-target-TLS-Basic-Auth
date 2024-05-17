# Secure Prometheus Monitoring Setup

This repository provides a guide for implementing authentication and encryption between Prometheus and its target (node exporter). Follow the steps below to set up a secure communication channel using TLS certificates and basic authentication.

## Target Setup

### 1. Generate the Certificate and the Key

Run the following command to generate a certificate and a key for the node exporter:

```sh
sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node.key -out node.crt -subj "/C=US/ST=California/L=Oakland/O=MyOrg/CN=localhost" -addext "subjectAltName = DNS:localhost"
```

### 2. Generate a Password for Basic Authentication

Install Apache utilities and create a password hash for basic authentication:

```sh
sudo apt install apache2-utils -y
htpasswd -nBC 12 ""
```

### 3. Create `config.yaml` for Node Exporter

Create the configuration file for node exporter at `/etc/node_exporter/config.yaml`:

```yaml
tls_server_config:
  cert_file: /etc/node_exporter/node.crt
  key_file: /etc/node_exporter/node.key
basic_auth_users:
  prometheus: $2y$12$yZDkIuKIpMAgoom3IAKDRuVZsOQfbuSXVy3LhoExWtWFmYMEvoAn6
```

### 4. Edit the Node Exporter Service File

Update the node exporter service file located at `/etc/systemd/system/node_exporter.service`:

```ini
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter --web.config.file="/etc/node_exporter/config.yaml"

[Install]
WantedBy=multi-user.target
```

### 5. Reload Daemon and Restart the Service

Reload the systemd daemon and restart the node exporter service:

```sh
sudo systemctl daemon-reload
sudo systemctl restart node_exporter
```

## Server Setup

### 1. Edit the Prometheus Configuration File

Edit the Prometheus configuration file at `/etc/prometheus/prometheus.yaml`. Ensure that the `node.crt` is copied to the server via SFTP and referenced correctly:

```yaml
- job_name: 'Linux Server'
  scheme: https
  basic_auth:
    username: prometheus
    password: pass
  tls_config:
    ca_file: "/etc/prometheus/node.crt"
    insecure_skip_verify: true
  static_configs:
    - targets: ['192.168.184.130:9100']
```

### 2. Restart Prometheus

Restart the Prometheus service and check the targets:

```sh
sudo systemctl restart prometheus
```
