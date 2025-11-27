# Practical-Exam-Module-3-IRONHACK
Dockerization of Java backend + Spring Boot + MySQL and Vue frontend

# 💻 Examen Práctico Módulo 3: Dockerización Full Stack

## 🎯 Objetivo

El objetivo de este proyecto es dockerizar una arquitectura de aplicación Full Stack (Backend Spring Boot, Base de Datos MySQL y Frontend Vue) y orquestar su comunicación utilizando comandos nativos de Docker y redes personalizadas.

---

## 🛠️ Requisitos y Arquitectura

### Servicios Contenedorizados

* **Backend:** Spring Boot (Java 17)
* **Base de Datos:** MySQL 8
* **Frontend:** Vue 3 (Servido por Nginx)

### Comunicación

Se utiliza una red de usuario de Docker para permitir la comunicación por DNS entre los servicios, garantizando el aislamiento de la base de datos y la correcta conectividad entre el Backend y el Frontend.

---

## 1. Tareas de Dockerización y Archivos Entregados

### 1.1 Dockerfile Backend (Spring Boot)

Se implementa un *Multistage Build* para reducir el tamaño de la imagen final, utilizando `maven` para la compilación y `eclipse-temurin:17-jre` para la ejecución.

* **Archivo:** `backend/Dockerfile`
    ```dockerfile
    # [Copia aquí el contenido completo del Dockerfile de tu Backend]
    ```

### 1.2 `application.properties` Modificado

El backend está configurado para leer las credenciales de la base de datos a través de variables de entorno, un requisito clave de la arquitectura de 12 factores.

* **Fragmento modificado:** `backend/src/main/resources/application.properties`
    ```properties
    spring.datasource.url=${DB_URL}
    spring.datasource.username=${DB_USERNAME}
    spring.datasource.password=${DB_PASSWORD}
    spring.jpa.hibernate.ddl-auto=update
    ```

### 1.3 Dockerfile Frontend (Vue + Nginx)

Se utiliza un *Multistage Build* que compila la aplicación Vue con `node:lts-alpine` y luego sirve los archivos estáticos resultantes utilizando una imagen ligera de `nginx:alpine`.

* **Archivo:** `frontend/Dockerfile`
    ```dockerfile
    # # --- STAGE 1: BUILD (Node.js) ---
FROM node:lts-alpine AS build-stage
WORKDIR /app

# Instala dependencias y construye la app Vue
COPY package*.json .
RUN npm install
COPY . .
# Se ejecuta el build, usando el .env para VITE_API_URL
RUN npm run build

# --- STAGE 2: SERVE (Nginx minimal) ---
FROM nginx:alpine AS production-stage

# Copia los archivos estáticos generados por el build
COPY --from=build-stage /app/dist /usr/share/nginx/html

# Puerto por defecto de Nginx
EXPOSE 80

# Mantiene Nginx en foreground
CMD ["nginx", "-g", "daemon off;"]
    ```

### 1.4 Archivo `.env` del Frontend

Configuración utilizada durante la etapa de *build* de Vue para especificar la URL base de la API.

* **Archivo:** `frontend/.env`
    ```dotenv
    VITE_API_URL=http://localhost:8080
    ```

---

## 2. Pasos de Ejecución Manual (Orquestación)

Para arrancar la pila completa, se sigue el siguiente orden, asegurando que todos los contenedores estén conectados a la red `examen-red`.

### PASO 0: Compilación y Preparación

Asegúrate de haber compilado el JAR de Spring Boot (`mvn clean package`) antes de construir la imagen del backend.

```bash
# 1. Compilar imagen del Backend
docker build -t mi-backend-java ./backend

# 2. Compilar imagen del Frontend
docker build -t mi-frontend-vue ./frontend
