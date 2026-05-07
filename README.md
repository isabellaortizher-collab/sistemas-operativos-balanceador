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

## 👩‍💻 Contribución individual — Isabella Ortiz Hernández

En este proyecto, mi responsabilidad principal fue la implementación de la **Parte 1: Clúster de servidores web con balanceo de carga**, encargándome de la infraestructura base del sistema.

### 🧩 Alcance de mi trabajo

### 1. Despliegue de infraestructura base
- Configuración de máquina virtual con Vagrant (Ubuntu 22.04)
- Instalación y verificación de Docker y Docker Compose
- Creación del entorno de trabajo del proyecto

---

### 2. Implementación de servidores backend
- Despliegue de 3 servidores web independientes usando Nginx en contenedores Docker:
  - backend1
  - backend2
  - backend3
- Configuración de páginas `index.html` para identificar cada nodo

📌 Objetivo: permitir identificar claramente qué servidor responde a cada petición.

---

### 3. Configuración del balanceador de carga
- Implementación de Apache HTTP Server como frontend
- Uso del módulo **mod_proxy_balancer**
- Configuración de proxy inverso hacia los 3 backends

---

### 4. Implementación de algoritmos de balanceo
Se configuraron y validaron dos estrategias:

- **Round Robin (byrequests)** → distribución equitativa de solicitudes
- **Least Connections (bybusyness)** → asignación al servidor con menor carga

---

### 5. Orquestación con Docker Compose
- Definición de servicios en `docker-compose.yml`
- Conexión de todos los contenedores en una misma red
- Exposición del balanceador en el puerto 8080

---

### 6. Pruebas funcionales del sistema
- Ejecución de pruebas con múltiples solicitudes HTTP
- Verificación de alternancia entre backend1, backend2 y backend3
- Validación del correcto funcionamiento del balanceador

---

## 🧠 Decisión de diseño

Se implementaron **3 contenedores backend independientes (backend1, backend2, backend3)** en lugar de escalar un único servicio.

---

## 🖥️ Entorno de trabajo

- Ubuntu 22.04 (Vagrant)
- IP servidor: `192.168.50.3`

---

Verificación:

```bash
docker --version
docker-compose --version
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
<h1>Backend X</h1>
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
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_http_module modules/mod_proxy_http.so

LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
LoadModule lbmethod_bybusyness_module modules/mod_lbmethod_bybusyness.so

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

---

## 🔄 Extensión del proyecto (integración futura del equipo)

Esta infraestructura queda preparada para la integración de la aplicación CybersecurityLab, pruebas de carga con Artillery y análisis de métricas por parte del equipo.

---

## 🛠️ Comandos útiles

```bash
sudo docker ps
sudo docker logs balanceador
sudo docker-compose down
```
---
## 🎯 Conclusión
Se implementó un clúster de servidores web con balanceo de carga funcional utilizando Apache y Docker, permitiendo la distribución de solicitudes entre múltiples nodos backend de forma eficiente y escalable.
