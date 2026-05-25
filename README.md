# 🐳 DataExplorer — Infraestructura Containerizada con Docker

Proyecto final de Sistemas Operativos — Universidad Autónoma de Occidente  
Implementación de una infraestructura containerizada con balanceo de carga, monitoreo con Prometheus y visualización con Grafana.

---

## 👥 Integrantes

- Isabella Ortiz Hernandez
- Isabela Cabezas Obregon
- Laura Victoria Gallo Payana

**Profesor:** Juan Manuel Alvarez Quinonez  
**Materia:** Sistemas Operativos

---

## 📋 Requisitos previos

Antes de comenzar, necesitas tener instalado en tu máquina física:

| Herramienta | Versión recomendada | Descarga |
|---|---|---|
| VirtualBox | 6.1 o superior | https://www.virtualbox.org |
| Vagrant | 2.3 o superior | https://www.vagrantup.com |
| Git | cualquiera | https://git-scm.com |

---

## 🖥️ Entorno Virtual (Vagrant)

El proyecto corre dentro de una máquina virtual Ubuntu 22.04 (Jammy) aprovisionada con Vagrant. El `Vagrantfile` define dos máquinas:

| Máquina | IP | RAM | CPUs | Rol |
|---|---|---|---|---|
| servidor | 192.168.50.3 | 2048 MB | 2 | Corre Docker y todos los contenedores |
| cliente | 192.168.50.2 | 1024 MB | 1 | Opcional, para pruebas de red |

### Vagrantfile

Crea un archivo llamado `Vagrantfile` en tu máquina con este contenido:

```ruby
Vagrant.configure("2") do |config|
  config.vm.boot_timeout = 600

  # =========================
  # SERVIDOR
  # =========================
  config.vm.define "servidor" do |servidor|
    servidor.vm.box = "ubuntu/jammy64"
    servidor.vm.hostname = "servidor"
    servidor.vm.network "private_network", ip: "192.168.50.3"
    servidor.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  # =========================
  # CLIENTE
  # =========================
  config.vm.define "cliente" do |cliente|
    cliente.vm.box = "ubuntu/jammy64"
    cliente.vm.hostname = "cliente"
    cliente.vm.network "private_network", ip: "192.168.50.2"
    cliente.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
  end
end
```

---

## 🚀 Paso a paso — Cómo desplegar el proyecto

### Paso 1 — Levantar la máquina virtual

En tu máquina física, abre una terminal en la carpeta donde tienes el `Vagrantfile` y corre:

```bash
vagrant up servidor
```

Espera a que termine (puede tomar varios minutos la primera vez mientras descarga la imagen de Ubuntu).

### Paso 2 — Conectarse a la VM

```bash
vagrant ssh servidor
```

### Paso 3 — Instalar Docker y Docker Compose dentro de la VM

```bash
# Actualizar paquetes
sudo apt update

# Instalar Docker
sudo apt install docker.io docker-compose -y

# Agregar tu usuario al grupo docker (para no usar sudo siempre)
sudo usermod -aG docker vagrant

# Verificar que Docker funciona
sudo docker --version
sudo docker-compose --version
```

### Paso 4 — Clonar el repositorio

```bash
# Crear y entrar a la carpeta del proyecto
mkdir proyecto1
cd proyecto1

# Clonar el repositorio
git clone https://github.com/isabellaortizher-collab/sistemas-operativos-balanceador.git .
```

> El punto `.` al final clona directamente en la carpeta actual sin crear una subcarpeta.

### Paso 5 — Levantar la infraestructura

```bash
sudo docker-compose up -d --build
```

Este comando construye las imágenes personalizadas y levanta los 8 contenedores en segundo plano. La primera vez puede tardar unos minutos.

### Paso 6 — Verificar que todo esté corriendo

```bash
sudo docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

Debes ver los 8 contenedores activos:

```
NAMES             IMAGE                            PORTS
balanceador       proyecto1_balanceador            0.0.0.0:8080->80/tcp
backend1          proyecto1_backend1               80/tcp
backend2          proyecto1_backend2               80/tcp
backend3          proyecto1_backend3               80/tcp
prometheus        prom/prometheus                  0.0.0.0:9090->9090/tcp
grafana           grafana/grafana                  0.0.0.0:3000->3000/tcp
apache-exporter   bitnami/apache-exporter:latest   0.0.0.0:9117->9117/tcp
node-exporter     prom/node-exporter:latest        0.0.0.0:9100->9100/tcp
```

### Paso 7 — Verificar el balanceo de carga

```bash
for i in {1..9}; do curl http://192.168.50.3:8080; echo ""; done
```

Salida esperada — rotación entre los 3 backends:

```
<h1>Backend 1</h1>
<h1>Backend 2</h1>
<h1>Backend 3</h1>
<h1>Backend 1</h1>
<h1>Backend 2</h1>
<h1>Backend 3</h1>
...
```

---

## 📐 Arquitectura

<img width="1411" height="770" alt="image" src="https://github.com/user-attachments/assets/455ab83d-aea1-4333-bbf8-9d46a91dae4b" />

---

## 📁 Estructura del Proyecto

```
proyecto1/
├── README.md
├── Vagrantfile
├── docker-compose.yml
├── prometheus.yml
├── backend1/
│   ├── Dockerfile
│   └── index.html
├── backend2/
│   ├── Dockerfile
│   └── index.html
├── backend3/
│   ├── Dockerfile
│   └── index.html
└── balanceador/
    ├── Dockerfile
    └── apache.conf
```

---

## 🔧 Descripción de los servicios

| Servicio | Imagen | Puerto | Descripción |
|---|---|---|---|
| balanceador | httpd:2.4 (custom) | 8080 | Proxy inverso, distribuye tráfico entre backends |
| backend1 | nginx:alpine (custom) | interno :80 | Instancia backend 1 |
| backend2 | nginx:alpine (custom) | interno :80 | Instancia backend 2 |
| backend3 | nginx:alpine (custom) | interno :80 | Instancia backend 3 |
| prometheus | prom/prometheus | 9090 | Recolección de métricas |
| grafana | grafana/grafana | 3000 | Visualización de métricas |
| apache-exporter | bitnami/apache-exporter | 9117 | Exporta métricas de Apache a Prometheus |
| node-exporter | prom/node-exporter | 9100 | Exporta métricas del sistema (CPU, RAM) |

### Red Docker
Todos los contenedores se comunican dentro de la red personalizada `red_balanceador` (driver: bridge), lo que permite resolución de nombres por servicio (ej. `http://backend1:80`).

### Volúmenes de persistencia
- `prometheus_data` — conserva los datos de métricas de Prometheus
- `grafana_data` — conserva dashboards y configuración de Grafana

---

## 📊 Monitoreo

### Prometheus
Accede desde tu navegador físico: `http://192.168.50.3:9090`

Ver targets activos: `http://192.168.50.3:9090/targets`

Targets configurados:
- `apache-exporter:9117` — métricas del balanceador Apache
- `node-exporter:9100` — métricas del sistema (CPU, RAM, red)

Intervalo de scraping: **5 segundos**

### Grafana
Accede desde tu navegador físico: `http://192.168.50.3:3000`

- **Usuario:** `admin`
- **Contraseña:** `admin`
- **Dashboard:** `DataExplorer - Infraestructura`

#### Paneles del dashboard

| Panel | Query PromQL | Qué muestra |
|---|---|---|
| Solicitudes HTTP/s | `rate(apache_accesses_total[1m])` | Tasa de requests por segundo al balanceador |
| Workers Apache | `apache_workers{state="busy/idle"}` | Workers ocupados vs libres |
| Uso de CPU (%) | `100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)` | Porcentaje de CPU del sistema |
| Memoria RAM (%) | `100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))` | Porcentaje de RAM usada |

<img width="1919" height="866" alt="image" src="https://github.com/user-attachments/assets/cf5e9c01-15dc-4be1-88b9-98043bac59a2" />

---

## 🧪 Pruebas de Carga con JMeter

Herramienta: **Apache JMeter 5.6.3** (ejecutado desde la máquina física Windows)

Configuración de la prueba:

| Parámetro | Valor |
|---|---|
| Hilos (usuarios concurrentes) | 50 |
| Período de subida (ramp-up) | 10 segundos |
| Iteraciones por hilo | 5 |
| Total de peticiones | 250 |
| URL objetivo | http://192.168.50.3:8080 |

Resultados obtenidos:

| Métrica | Valor |
|---|---|
| Peticiones totales | 250 |
| Tiempo medio de respuesta | 4 ms |
| Tiempo mínimo | 1 ms |
| Tiempo máximo | 44 ms |
| Tasa de error | 0.00% |
| Rendimiento (throughput) | 25.6 req/seg |

<img width="1919" height="501" alt="image" src="https://github.com/user-attachments/assets/fa59c3dd-202c-4450-926c-f671d0f2dddb" />

---

## 🛑 Apagar la infraestructura

```bash
# Detener y eliminar los contenedores
sudo docker-compose down

# Apagar la VM desde tu máquina física
vagrant halt servidor
```

---

## 🔗 Recursos

- Documentación completa del proyecto (PDF): adjunta en la entrega
- Repositorio: https://github.com/isabellaortizher-collab/sistemas-operativos-balanceador.git
