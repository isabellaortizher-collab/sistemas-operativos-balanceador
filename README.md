# 🐳 DataExplorer — Infraestructura Containerizada con Docker

Proyecto final de Sistemas Operativos. Implementación de una infraestructura containerizada con balanceo de carga, monitoreo con Prometheus y visualización con Grafana.

---

## 📐 Arquitectura

<img width="1411" height="770" alt="image" src="https://github.com/user-attachments/assets/455ab83d-aea1-4333-bbf8-9d46a91dae4b" />

---

## 📁 Estructura del Proyecto

```
proyecto1/
├── README.md
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

## 🚀 Servicios

| Servicio | Imagen | Puerto | Descripción |
|---|---|---|---|
| balanceador | httpd:2.4 (custom) | 8080 | Proxy inverso con balanceo de carga |
| backend1 | nginx:alpine (custom) | — | Instancia backend 1 |
| backend2 | nginx:alpine (custom) | — | Instancia backend 2 |
| backend3 | nginx:alpine (custom) | — | Instancia backend 3 |
| prometheus | prom/prometheus | 9090 | Recolección de métricas |
| grafana | grafana/grafana | 3000 | Visualización de métricas |
| apache-exporter | bitnami/apache-exporter | 9117 | Exportador de métricas Apache |
| node-exporter | prom/node-exporter | 9100 | Métricas del sistema |

---

## ⚙️ Configuración

### docker-compose.yml

Define todos los servicios, la red personalizada `red_balanceador` (bridge) y los volúmenes de persistencia para Prometheus y Grafana.

### balanceador/apache.conf

Configura Apache como proxy inverso con balanceo de carga hacia los 3 backends. Incluye:
- Módulos: `mod_proxy`, `mod_proxy_balancer`, `mod_status`
- Algoritmo: `bybusyness` (Least Connections)
- Ruta `/server-status` para scraping de métricas

### prometheus.yml

Define los targets monitoreados:
- `apache-exporter:9117` — métricas del balanceador Apache
- `node-exporter:9100` — métricas del sistema (CPU, RAM)

Intervalo de scraping: **5 segundos**

---

## 🏃 Cómo ejecutar

```bash
# Clonar el repositorio
git clone https://github.com/isabellaortizher-collab/sistemas-operativos-balanceador.git
cd proyecto1

# Levantar toda la infraestructura
sudo docker-compose up -d --build

# Verificar que todos los contenedores estén corriendo
sudo docker ps
```

### Verificar balanceo de carga

```bash
for i in {1..9}; do curl http://localhost:8080; echo ""; done
```

Salida esperada (rotación entre backends):
```
<h1>Backend 1</h1>
<h1>Backend 2</h1>
<h1>Backend 3</h1>
...
```

---

## 📊 Monitoreo

### Prometheus
- URL: `http://localhost:9090`
- Targets: `http://localhost:9090/targets`

### Grafana
- URL: `http://localhost:3000`
- Usuario: `admin` / Contraseña: `admin`
- Dashboard: **DataExplorer - Infraestructura**

#### Métricas monitoreadas

| Panel | Query PromQL | Descripción |
|---|---|---|
| Solicitudes HTTP/s | `rate(apache_accesses_total[1m])` | Tasa de requests al balanceador |
| Workers Apache | `apache_workers{state="busy/idle"}` | Workers ocupados vs libres |
| Uso de CPU | `100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)` | % CPU del sistema |
| Memoria RAM | `100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))` | % RAM usada |
<img width="1919" height="866" alt="image" src="https://github.com/user-attachments/assets/cf5e9c01-15dc-4be1-88b9-98043bac59a2" />


---

## 🧪 Pruebas de Carga (JMeter)

Herramienta: Apache JMeter 5.6.3

Configuración:
- **Hilos (usuarios):** 50
- **Período de subida:** 10 segundos
- **Iteraciones:** 5
- **Total de peticiones:** 250

Resultados:

| Métrica | Valor |
|---|---|
| Peticiones totales | 250 |
| Tiempo medio de respuesta | 4 ms |
| Tiempo mínimo | 1 ms |
| Tiempo máximo | 44 ms |
| % de errores | 0.00% |
| Rendimiento | 25.6 req/seg |

<img width="1919" height="501" alt="image" src="https://github.com/user-attachments/assets/fa59c3dd-202c-4450-926c-f671d0f2dddb" />

---

## 🌐 Red Docker

Red personalizada: `red_balanceador` (driver: bridge)

Todos los contenedores se comunican entre sí por nombre de servicio dentro de esta red.

---

## 👥 Integrantes del grupo

- Isabella Ortiz 
- Isabela Cabezas
- Laura Gallo
