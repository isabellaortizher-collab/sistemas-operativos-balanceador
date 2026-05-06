# 🖥️ PROYECTO 1 — Balanceador de Carga con Apache y Docker
Proyecto de Curso 2026-1 · Universidad Autónoma de Occidente · Servicios Telemáticos

---

## 👩‍💻 Equipo de trabajo

| Integrante               | Rol en el proyecto |
|------------------------|------------------|
| Isabella Ortiz         | Infraestructura base + Docker + Balanceador |
| Julián Viafara         | Integración aplicación web (CybersecurityLab) |
| Samuel Sepúlveda       | Configuración de algoritmos de balanceo |
| Sebastián Cobos        | Pruebas de carga (Artillery) |
| Isabella Cabezas       | Métricas y análisis de resultados |
| Valentina Velastegui   | Documentación (README + informe IEEE) |

---

## 📌 ¿Qué se hizo?

Se construyó la **infraestructura base del sistema**:

- Máquina virtual con Vagrant
- Instalación de Docker y Docker Compose
- 3 servidores backend con Nginx
- 1 balanceador de carga con Apache
- Orquestación con Docker Compose

---

## 🖥️ Entorno de trabajo

- Ubuntu 22.04 (Vagrant)
- IP servidor: `192.168.50.3`

---

## ⚙️ Instalación

Dentro de la VM:

```bash
vagrant ssh servidor
sudo apt update
sudo apt install -y docker.io docker-compose
```

Verificación:

```bash
docker --version
docker-compose --version
```

---

## 📁 Creación del proyecto

```bash
mkdir proyecto1
cd proyecto1
```

---

## 🧩 Arquitectura
Cliente → Balanceador (Apache) → Backend1 / Backend2 / Backend3

---

## 📁 Estructura
```
.
│
├── docker-compose.yml
├── README.md
│
├── balanceador/
│   ├── Dockerfile
│   └── apache.conf
│
├── backend1/
├── backend2/
├── backend3/
│   ├── Dockerfile
│   └── index.html
```
---
## 📄 Explicación de archivos

- docker-compose.yml → define todos los servicios (backends y balanceador)
- apache.conf → configuración del balanceador de carga
- backend*/Dockerfile → crea cada servidor web con Nginx
- index.html → contenido de prueba de cada backend
---
## 🧩 Backends (Nginx)

**Dockerfile**

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

**index.html** (ejemplo)

```html
<h1>Backend 1</h1>
```

---

## ⚖️ Balanceador (Apache)

**Dockerfile**

```dockerfile
FROM httpd:2.4
COPY apache.conf /usr/local/apache2/conf/httpd.conf
```

**apache.conf**

```apache
LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so

Listen 80

<Proxy "balancer://cluster">
    BalancerMember http://backend1:80
    BalancerMember http://backend2:80
    BalancerMember http://backend3:80
    ProxySet lbmethod=byrequests
</Proxy>

ProxyPass "/" "balancer://cluster/"
ProxyPassReverse "/" "balancer://cluster/"
```

---

## 🐳 docker-compose.yml

```yaml
version: '3'
services:
  backend1:
    build: ./backend1
  backend2:
    build: ./backend2
  backend3:
    build: ./backend3
  balanceador:
    build: ./balanceador
    ports:
      - "8080:80"
```

---

## ▶️ Ejecución

```bash
sudo docker-compose up --build
```

---

## 🌐 Acceso
http://192.168.50.3:8080

---

## 🔁 Algoritmos de balanceo implementados

Se configuraron dos algoritmos de balanceo en Apache:

### 1. Round Robin (por defecto)
Distribuye las peticiones de manera equitativa entre los servidores backend.

```apache
ProxySet lbmethod=byrequests
```

---

### 2. Least Connections
Envía las peticiones al servidor con menos conexiones activas.

```apache
ProxySet lbmethod=bybusyness
```

---

### 🔄 Cómo cambiar de algoritmo

Editar el archivo:

```bash
nano balanceador/apache.conf
```

Y cambiar la línea:

```apache
ProxySet lbmethod=...
```

Luego reiniciar:

```bash
sudo docker-compose down
sudo docker-compose up --build
```

---

### 🎯 Propósito

Esto permite comparar el comportamiento del sistema bajo diferentes estrategias de balanceo, como lo requiere el proyecto.

---

## 🧪 Verificación

- Recargar la página varias veces
- Debe cambiar entre backend1, backend2 y backend3

---

## 🧪 Prueba de balanceo

Para verificar el funcionamiento del balanceador sin interferencia de caché del navegador:

```bash
for i in {1..10}; do curl http://192.168.50.3:8080; echo ""; done
```

Resultado esperado:

- Respuestas alternando entre:
  - Backend 1
  - Backend 2
  - Backend 3

Esto confirma que el balanceador distribuye correctamente las peticiones.

## ✅ Estado actual

- ✔ Balanceador funcionando
- ✔ 3 backends activos
- ✔ Docker funcionando

---

## ⚠️ Pendiente (equipo)

### 🔴 Integrar aplicación real
https://github.com/julianviafara-arch/CybersecurityLab

### 🔴 Cambiar algoritmo
Modificar:

```apache
ProxySet lbmethod=byrequests
```

### 🔴 Pruebas de carga
Usar Artillery

### 🔴 Métricas
- Latencia
- Throughput
- Errores

### 🔴 Documentación
- Informe IEEE

---

## 🛠️ Comandos útiles

```bash
sudo docker ps
sudo docker logs balanceador
sudo docker-compose down
```
---
## 🎯 Conclusión
Se dejó lista la infraestructura base con balanceo de carga funcional, preparada para integrar una aplicación real y realizar pruebas de rendimiento.
