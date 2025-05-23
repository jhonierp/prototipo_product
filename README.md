# Documentación Completa de Docker: De Principiante a Intermedio

## 1. Creación de una Imagen Personalizada con Nginx

### Paso 1: Crear un archivo Dockerfile
```dockerfile
FROM nginx:alpine          # Usa la imagen oficial de Nginx basada en Alpine Linux (ligera)
COPY ./html /usr/share/nginx/html   # Copia archivos locales al directorio de contenido de Nginx
EXPOSE 80                  # Documenta que el contenedor escuchará en el puerto 80
```

**Explicación:**
- `FROM`: Define la imagen base que usaremos (Nginx con Alpine Linux)
- `COPY`: Transfiere archivos desde el host al contenedor
- `EXPOSE`: Documenta qué puertos estarán disponibles (no los abre automáticamente)

### Paso 2: Crear un archivo index.html
```html
<!-- html/index.html -->
<!DOCTYPE html>
<html>
  <head><title>Hola desde Docker</title></head>
  <body><h1>¡Funciona!</h1></body>
</html>
```

**Nota importante:** Debes crear un directorio llamado `html` en el mismo nivel que tu Dockerfile y guardar el archivo index.html dentro de él.

### Paso 3: Construir la imagen
```bash
docker build -t mi-nginx .
```

**Explicación:**
- `docker build`: Comando para crear una nueva imagen
- `-t mi-nginx`: Asigna un nombre (tag) a la imagen
- `.`: Usa el Dockerfile en el directorio actual

### Paso 4: Ejecutar el contenedor
```bash
docker run -d -p 8080:80 mi-nginx
```

**Explicación:**
- `docker run`: Inicia un nuevo contenedor
- `-d`: Modo detached (en segundo plano)
- `-p 8080:80`: Mapea el puerto 8080 del host al puerto 80 del contenedor
- `mi-nginx`: Nombre de la imagen a usar

**Verificación:** Abre tu navegador y visita http://localhost:8080

**Comandos útiles adicionales:**
```bash
docker ps                  # Ver contenedores en ejecución
docker stop [CONTAINER_ID] # Detener un contenedor
docker rm [CONTAINER_ID]   # Eliminar un contenedor
docker images              # Listar imágenes disponibles
docker logs [CONTAINER_ID] # Ver logs del contenedor
```

## 2. Ejercicio Guiado: Aplicación Node.js con Express

### Estructura de archivos
```
proyecto-node/
│
├── app.js           # Aplicación principal de Node.js
├── package.json     # Dependencias y configuración
└── Dockerfile       # Instrucciones para construir la imagen
```

### Archivos base:

#### app.js
```javascript
const express = require('express');
const app = express();

// Ruta principal que devuelve un mensaje simple
app.get('/', (req, res) => res.send('Hola desde Node y Docker!'));

// Inicia el servidor en el puerto 3000
app.listen(3000, () => console.log('Servidor en puerto 3000'));
```

#### package.json
```json
{
  "name": "app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2"
  },
  "scripts": {
    "start": "node app.js"
  }
}
```

#### Dockerfile
```dockerfile
FROM node:18           # Usa la imagen oficial de Node.js versión 18
WORKDIR /app           # Establece el directorio de trabajo dentro del contenedor

# Copia los archivos de dependencias primero (aprovecha el caché de Docker)
COPY package*.json ./  
RUN npm install        # Instala las dependencias

# Copia el resto de archivos de la aplicación
COPY . .               
EXPOSE 3000            # Documenta que la app usa el puerto 3000
CMD ["npm", "start"]   # Comando para iniciar la aplicación
```

**Explicación de buenas prácticas:**
1. Copiamos primero solo los archivos package.json antes de ejecutar npm install
2. Esto aprovecha el sistema de capas y caché de Docker, evitando reinstalar dependencias si solo cambia el código fuente
3. Establecemos un WORKDIR para evitar conflictos de rutas

### Comandos para construir y ejecutar:
```bash
docker build -t app-node .     # Construye la imagen con el nombre "app-node"
docker run -d -p 3000:3000 app-node  # Ejecuta el contenedor mapeando el puerto 3000
```

**Verificación:** Abre tu navegador y visita http://localhost:3000

## 3. Ejercicios Adicionales (Con Soluciones)

### 3.1. Crea una imagen de Python que ejecute un script llamado main.py

**Estructura de archivos:**
```
proyecto-python/
│
├── main.py        # Script de Python a ejecutar
└── Dockerfile     # Instrucciones para la imagen
```

**main.py:**
```python
# Un script simple que imprime un mensaje
print("¡Hola desde Python en Docker!")

# Demo de una función simple
def sumar(a, b):
    return a + b

resultado = sumar(5, 3)
print(f"5 + 3 = {resultado}")
```

**Dockerfile:**
```dockerfile
FROM python:3.9-slim   # Usamos una imagen base ligera de Python

WORKDIR /app           # Establecemos directorio de trabajo
COPY main.py .         # Copiamos nuestro script

# Comando para ejecutar el script
CMD ["python", "main.py"]
```

**Comandos:**
```bash
docker build -t mi-python .            # Construye la imagen
docker run mi-python                   # Ejecuta el contenedor
```

### 3.2. Agregar variables de entorno (ENV) y mostrarlas en la terminal

**Estructura de archivos:**
```
proyecto-env/
│
├── mostrar_env.py     # Script para mostrar variables
└── Dockerfile         # Con definición de variables
```

**mostrar_env.py:**
```python
import os

print("Variables de entorno configuradas:")
print(f"NOMBRE: {os.environ.get('NOMBRE', 'No definido')}")
print(f"ENTORNO: {os.environ.get('ENTORNO', 'No definido')}")
print(f"VERSION: {os.environ.get('VERSION', 'No definido')}")
```

**Dockerfile:**
```dockerfile
FROM python:3.9-slim

# Definimos variables de entorno
ENV NOMBRE="Usuario Docker"
ENV ENTORNO="Desarrollo" 
ENV VERSION="1.0"

WORKDIR /app
COPY mostrar_env.py .

CMD ["python", "mostrar_env.py"]
```

**Comandos:**
```bash
docker build -t python-env .            # Construye la imagen
docker run python-env                   # Ejecuta con variables predefinidas

# Sobrescribir variables al ejecutar:
docker run -e NOMBRE="Otro Usuario" -e ENTORNO="Producción" python-env
```

### 3.3. Usar ENTRYPOINT en vez de CMD

**¿Cuál es la diferencia?**
- `CMD`: Define comandos y argumentos por defecto que pueden ser sobrescritos
- `ENTRYPOINT`: Define un ejecutable fijo que siempre se ejecutará (los argumentos de docker run se añaden)

**Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY saludo.py .

# Definimos el ejecutable principal que no puede ser sobrescrito
ENTRYPOINT ["python", "saludo.py"]

# Argumentos por defecto que pueden ser sobrescritos
CMD ["Mundo"]
```

**saludo.py:**
```python
import sys

# Toma el primer argumento o usa "Amigo" como valor predeterminado
nombre = sys.argv[1] if len(sys.argv) > 1 else "Amigo"
print(f"¡Hola, {nombre}!")
```

**Comandos:**
```bash
docker build -t python-entrypoint .     # Construye la imagen
docker run python-entrypoint            # Ejecuta con argumento por defecto: "Mundo"
docker run python-entrypoint "Docker"   # Sobrescribe argumento: "Docker"
```

## 4. Ejercicio Final: Imagen Personalizada con PHP y Apache

### Objetivo
Crear una imagen personalizada que:
* Use como base `php:apache`
* Copie una página PHP a `/var/www/html`
* Exponga el puerto 80
* Personalice el mensaje con tu nombre

### Estructura de archivos
```
proyecto-php/
│
├── index.php       # Página PHP personalizada
└── Dockerfile      # Configuración de la imagen
```

### index.php
```php
<!DOCTYPE html>
<html>
<head>
    <title>Mi Página PHP con Docker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 40px;
            line-height: 1.6;
            color: #333;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 5px;
            background-color: #f9f9f9;
        }
        h1 {
            color: #2c3e50;
        }
        .info {
            background-color: #e8f4f8;
            padding: 15px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hola desde PHP y Docker</h1>
        
        <div class="info">
            <p><strong>Nombre:</strong> [TU NOMBRE]</p>
            <p><strong>Fecha y hora actual:</strong> <?php echo date('Y-m-d H:i:s'); ?></p>
            <p><strong>Versión de PHP:</strong> <?php echo phpversion(); ?></p>
            <p><strong>Información del servidor:</strong> <?php echo $_SERVER['SERVER_SOFTWARE']; ?></p>
        </div>
        
        <h2>Variables de Entorno</h2>
        <pre><?php print_r($_ENV); ?></pre>
    </div>
</body>
</html>
```

### Dockerfile
```dockerfile
FROM php:apache                        # Usa la imagen oficial de PHP con Apache

# Instala extensiones PHP útiles (opcional)
RUN docker-php-ext-install pdo pdo_mysql

# Copia el archivo PHP personalizado al directorio web
COPY index.php /var/www/html/

# Variables de entorno personalizadas
ENV CREADOR="[TU NOMBRE]" 
ENV APP_VERSION="1.0"

# El puerto 80 ya está expuesto en la imagen base, pero lo documentamos explícitamente
EXPOSE 80

# Apache se inicia automáticamente, no necesitamos CMD adicional
```

### Comandos para construir y ejecutar
```bash
# Construir la imagen
docker build -t mi-php-app .

# Ejecutar el contenedor
docker run -d -p 8080:80 mi-php-app

# Verificar en el navegador
# Abre http://localhost:8080
```

## 5. Conceptos Adicionales Importantes

### Volúmenes en Docker
Los volúmenes permiten persistir datos fuera del ciclo de vida del contenedor:

```bash
# Crear un volumen
docker volume create mi-volumen

# Usar un volumen al iniciar un contenedor
docker run -d -p 8080:80 -v mi-volumen:/var/www/html mi-nginx

# Montar un directorio del host
docker run -d -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html mi-nginx
```

### Docker Compose
Docker Compose permite definir y ejecutar aplicaciones multi-contenedor:

**docker-compose.yml** (ejemplo simple):
```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "8080:80"
  
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: ejemplo
      MYSQL_DATABASE: app_db
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

**Comandos:**
```bash
docker-compose up -d    # Inicia todos los servicios
docker-compose down     # Detiene y elimina contenedores
docker-compose logs     # Muestra logs de todos los servicios
```

### Redes en Docker
Para comunicación entre contenedores:

```bash
# Crear una red
docker network create mi-red

# Conectar contenedores a la red
docker run -d --name web --network mi-red nginx
docker run -d --name db --network mi-red mysql

# Los contenedores en la misma red pueden comunicarse por nombre
# Por ejemplo, 'web' podría conectarse a 'db' usando el hostname 'db'
```

## 6. Buenas Prácticas en Docker

1. **Imágenes ligeras**: Usa Alpine o Slim cuando sea posible
2. **Principio de una sola responsabilidad**: Un contenedor, un servicio
3. **Aprovecha el caché**: Ordena las instrucciones del Dockerfile de menos a más cambiantes
4. **No ejecutes como root**: Usa USER para cambiar a un usuario no privilegiado
5. **Multi-stage builds**: Para separar dependencias de compilación y ejecución
6. **Etiqueta las imágenes**: No uses siempre `:latest`
7. **Usa .dockerignore**: Para excluir archivos innecesarios
8. **Variables de configuración**: Usa variables de entorno para configuración

## 7. Recursos Adicionales

- [Documentación oficial de Docker](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/) - Repositorio de imágenes
- [Play with Docker](https://labs.play-with-docker.com/) - Entorno para practicar
- [Docker Curriculum](https://docker-curriculum.com/) - Tutorial completo
