# 📝 Informe de Práctica: Dockerización y Configuración de Pipeline de la Aplicación de Rick and Morty 🚀

## 🌟 Introducción

El objetivo de esta práctica fue **dockerizar** una aplicación de **Rick and Morty** y configurar un **pipeline de Continuous Delivery (CD)** utilizando **GitHub Actions**. Este proceso nos permitió aprender sobre la creación de **contenedores Docker** y la **automatización de la entrega de aplicaciones**, lo cual es fundamental en el desarrollo moderno de software. 💻✨

## 🐳 Proceso de Dockerización

### 1. 🛠️ Creación del Dockerfile

Para iniciar el proceso de dockerización, se construyó un `Dockerfile` que permite crear una imagen de la aplicación. Este archivo contiene las instrucciones necesarias para:

- **Definir la imagen base**: Se eligió `node:18-alpine` como base para la construcción de la aplicación. 📦
- **Configurar el entorno de trabajo**: Se estableció el directorio de trabajo en `/app`. 🗂️
- **Instalar dependencias**: Se copió el archivo `package*.json` y se ejecutó `npm install` para instalar las dependencias necesarias. 📥
- **Construir la aplicación**: Se copiaron los archivos de la aplicación y se ejecutó `npm run build` para generar la versión de producción. ⚙️
- **Servir la aplicación**: Se utilizó `nginx:alpine` para servir los archivos construidos. 🌐

El contenido del `Dockerfile` es el siguiente:

```dockerfile
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## 🔧 Configuración del Pipeline de Continuous Delivery

### 1. 🏗️ Configuración Inicial

Se configuró un pipeline utilizando **GitHub Actions** para automatizar el proceso de construcción y entrega de la imagen Docker. Inicialmente, se intentó utilizar **Docker Hub**, pero se optó por **GitHub Container Registry** debido a una mejor integración con el repositorio. 🎉

### 2. 📄 Creación del Archivo de Configuración

El archivo de configuración del pipeline (`.github/workflows/docker.yml`) se diseñó para realizar los siguientes pasos:

- **Realizar el checkout del repositorio**: Obtener el código fuente del repositorio. 📥
- **Configurar Docker Buildx**: Preparar el entorno para construir imágenes Docker. 🛠️
- **Iniciar sesión en el GitHub Container Registry**: Autenticarse para poder subir imágenes. 🔑
- **Construir y enviar la imagen de Docker**: Crear la imagen y enviarla al registro. 🚢

Además, se utilizó **Docker Buildx**, una extensión de Docker que permite construir imágenes de manera más eficiente y flexible. Proporciona soporte para la construcción **multiplataforma**, **caché avanzado** y comandos extensibles, mejorando el rendimiento y la flexibilidad del proceso de creación de imágenes. Su integración con **GitHub Actions** facilita la automatización del proceso de construcción y entrega en pipelines de CD. 🤖✨

El contenido del archivo de configuración es el siguiente:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # Paso 1: Checkout del repositorio
      - name: Checkout code
        uses: actions/checkout@v3

      # Paso 2: Configurar Docker Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      # Paso 3: Iniciar sesión en GitHub Container Registry
      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Paso 4: Configurar el nombre del actor en minúsculas
      - name: Set actor name to lowercase
        id: set_actor
        run: echo "lowercase_actor=$(echo $GITHUB_ACTOR | tr '[:upper:]' '[:lower:]')" >> $GITHUB_ENV

      # Paso 5: Construir y enviar la imagen de Docker
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ghcr.io/${{ env.lowercase_actor }}/docker-test:latest
```

### 3. 📈 Resultados y Aprendizajes

La implementación del pipeline permitió **automatizar** el proceso de construcción y entrega de la aplicación. Aunque se enfrentaron algunos problemas iniciales al intentar utilizar **Docker Hub**, la decisión de migrar a **GitHub Container Registry** simplificó la integración. 🌟

![image](https://github.com/user-attachments/assets/117d5d7b-8b8a-4141-8884-2da42a4f19dc)

---
