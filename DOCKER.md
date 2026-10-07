# React Example Containers

Aplicación React preparada para ejecutarse dentro de un contenedor Docker usando un **multi-stage build**.

La aplicación se compila con Node.js y posteriormente los archivos estáticos se sirven utilizando Nginx, manteniendo una imagen final ligera.

## Requisitos

* Docker
* Docker Desktop (Windows/macOS)

Verifica que Docker esté disponible:

```bash
docker --version
```

## Estructura

```text
react-example-containers/
├── src/
├── public/
├── package.json
├── package-lock.json
├── Dockerfile
├── .dockerignore
└── README.md
```

## Dockerfile

El proyecto utiliza un build de dos etapas:

```dockerfile
# ---------- Build ----------
FROM node:22-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

RUN npm run build


# ---------- Runtime ----------
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### ¿Qué hace?

**1. Builder**

Utiliza Node.js para instalar las dependencias y compilar React:

```text
node:22-alpine
       ↓
npm ci
       ↓
npm run build
       ↓
/app/dist
```

**2. Runtime**

El resultado de la compilación se copia a Nginx:

```text
/app/dist
    ↓
/usr/share/nginx/html
    ↓
nginx:alpine
```

El contenedor final no necesita Node.js ni `node_modules`.

---

# Crear la imagen Docker

Desde la raíz del proyecto ejecuta:

```bash
docker build -t mi-react-app .
```

Puedes verificar que la imagen fue creada:

```bash
docker images
```

Deberías encontrar:

```text
mi-react-app
```

---

# Crear y levantar el contenedor

Ejecuta:

```bash
docker run -d \
  --name mi-react-app \
  -p 8080:80 \
  mi-react-app
```

En Windows PowerShell también puedes utilizarlo en una sola línea:

```bash
docker run -d --name mi-react-app -p 8080:80 mi-react-app
```

La aplicación estará disponible en:

```text
http://localhost:8080
```

---

# Ver los contenedores

Para ver los contenedores ejecutándose:

```bash
docker ps
```

Ejemplo:

```text
CONTAINER ID   IMAGE          PORTS                  NAMES
xxxxxxxxxxxx   mi-react-app   0.0.0.0:8080->80/tcp   mi-react-app
```

---

# Ver los logs

Para consultar los logs de Nginx:

```bash
docker logs mi-react-app
```

También puedes seguir los logs en tiempo real:

```bash
docker logs -f mi-react-app
```

Presiona `Ctrl + C` para salir.

---

# Detener el contenedor

```bash
docker stop mi-react-app
```

# Volver a iniciar el contenedor

Si el contenedor ya existe:

```bash
docker start mi-react-app
```

No necesitas volver a ejecutar `docker build` mientras no hayas cambiado la imagen.

---

# Eliminar el contenedor

Primero detenlo:

```bash
docker stop mi-react-app
```

Después:

```bash
docker rm mi-react-app
```

---

# Crear una nueva versión

Cuando hagas cambios en el código:

### 1. Construir nuevamente la imagen

```bash
docker build -t mi-react-app .
```

### 2. Eliminar el contenedor anterior

```bash
docker stop mi-react-app
docker rm mi-react-app
```

### 3. Crear el nuevo contenedor

```bash
docker run -d --name mi-react-app -p 8080:80 mi-react-app
```

---

# Flujo completo

Después de modificar el proyecto, el flujo normal será:

```bash
docker build -t mi-react-app .
docker stop mi-react-app
docker rm mi-react-app
docker run -d --name mi-react-app -p 8080:80 mi-react-app
```

Después abre:

```text
http://localhost:8080
```

---

# Comandos útiles

### Ver contenedores activos

```bash
docker ps
```

### Ver todos los contenedores

```bash
docker ps -a
```

### Ver imágenes

```bash
docker images
```

### Detener

```bash
docker stop mi-react-app
```

### Iniciar

```bash
docker start mi-react-app
```

### Reiniciar

```bash
docker restart mi-react-app
```

### Eliminar contenedor

```bash
docker rm mi-react-app
```

### Eliminar imagen

```bash
docker rmi mi-react-app
```

---

# .dockerignore

Para evitar copiar archivos innecesarios al contexto de Docker, utiliza:

```dockerignore
node_modules
dist
.git
.gitignore
Dockerfile
docker-compose.yml
npm-debug.log
yarn-error.log
.env
.env.*
```

## Arquitectura

```text
                 Docker Build
                      │
                      ▼
             ┌─────────────────┐
             │  Node 22 Alpine │
             │                 │
             │  npm ci         │
             │  npm run build  │
             └────────┬────────┘
                      │
                      │ dist/
                      ▼
             ┌─────────────────┐
             │   Nginx Alpine  │
             │                 │
             │  React SPA      │
             │  HTML / JS / CSS│
             └────────┬────────┘
                      │
                      │ Port 80
                      ▼
                localhost:8080
```

## Nota

Este proyecto está pensado para **producción**.

Para desarrollo local con Hot Reload es preferible ejecutar React directamente con:

```bash
npm run dev
```

y utilizar Docker para generar y probar el build de producción.
