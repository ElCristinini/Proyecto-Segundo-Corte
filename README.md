# Proyecto de Monitoreo de Máquinas Virtuales con Prometheus y Grafana

Este proyecto implementa una solución de monitoreo distribuido usando **Prometheus**, **Grafana** y **Node Exporter** para observar métricas de múltiples máquinas virtuales en un entorno local con red en modo bridge (`red_br0`).

---

## 📦 Componentes del Proyecto

- **Prometheus**: Recolector de métricas.
- **Grafana**: Visualización de datos.
- **Node Exporter**: Exporta métricas de las VMs.
- **Docker + Docker Compose**: Orquestación y contenedores.
- **Red en modo bridge** (`red_br0`): Permite visibilidad directa entre contenedores y VMs.

---

## 🛠️ Instalación de Node Exporter en VMs

### Requisitos:
- Acceso a internet en cada VM.
- Usuario con permisos de superusuario.

### Pasos:
```bash
# Como root o usando sudo
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
chmod +x /usr/local/bin/node_exporter
```

### Servicio systemd para que se inicie automáticamente:
```ini
# /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter
After=network.target

[Service]
ExecStart=/usr/local/bin/node_exporter --web.listen-address=0.0.0.0:9100
User=nobody
Restart=always

[Install]
WantedBy=default.target
```

```bash
systemctl daemon-reexec
systemctl daemon-reload
systemctl enable node_exporter
systemctl start node_exporter
```

---

## 🧱 Configuración de Prometheus

El archivo `prometheus.yml` contiene las IPs de todas las VMs:

```yaml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets:
          - '192.168.100.11:9100'  # Manjaro
          - '192.168.100.12:9100'  # Rocky
          - '192.168.100.13:9100'  # Arch
          - '192.168.100.21:9100'  # Garuda
          - '192.168.100.22:9100'  # Alpine
          - '192.168.100.23:9100'  # Debian
          - '192.168.100.50:9100'  # Central
          - '192.168.100.61:9100'  # Central2
```

---

## 📊 Configuración de Grafana

- Fuente de datos: Prometheus (`http://192.168.100.60:9090`)
- Dashboard: Se utilizó uno con visualización de:
  - CPU
  - RAM
  - SWAP
  - Red
  - Disco

Variables creadas:
- `job`
- `nodename`
- `node` (con instancias)
- `diskdevices`

---

## 🚀 Ejecución con Docker Compose

```bash
docker compose up -d
```

Esto levanta Prometheus y Grafana conectados a la red `red_br0`.

---

## ❤️ Créditos

Este proyecto fue configurado y desplegado con esfuerzo, persistencia y muchas pruebas, gracias al profe por la asesoria en horario no laboral
