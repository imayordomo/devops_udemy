# Docker - Apuntes

## 1. Instalar Docker Engine en Ubuntu

Documentación oficial:

https://docs.docker.com/engine/install/ubuntu/
---

## 2. Configuración inicial

### Añadir usuario al grupo Docker

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

> Puede ser necesario cerrar sesión y volver a entrar para que los cambios surtan efecto.

---

## 3. Rotación de logs

Los contenedores pueden generar muchos logs. Para limitar el espacio utilizado:

```bash
sudo nano /etc/docker/daemon.json
```

Configuración para mantener un máximo de 3 ficheros de log de 10 MB cada uno:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Aplicar cambios:

```bash
sudo systemctl restart docker
```

---

## 4. Ver contenedores

Ver todos los contenedores:

```bash
docker ps -a
```

Sin `-a` solo muestra los contenedores en ejecución:

```bash
docker ps
```

---

## 5. Recomendaciones

Para evitar errores y consultar comandos frecuentes:

```text
commands.md
```

---

## 6. Comprobar puertos

Ver si un puerto está ocupado:

```bash
sudo lsof -i -P -n | grep 9000
```

Verificar conectividad con un puerto:

```bash
telnet localhost 8080
```

---

## 7. Ver archivos con permisos

Linux:

```bash
ls -la
```

Windows:

```cmd
dir
```

---

## 8. Ver imágenes Docker

```bash
docker image ls
```

---

## 9. Eliminar imágenes o contenedores

Eliminar un contenedor:

```bash
docker container rm ID
```

Eliminar una imagen:

```bash
docker image rm ID
```

> Hay que eliminar primero el contenedor que esté utilizando una imagen antes de poder eliminar dicha imagen.

Eliminar recursos no utilizados:

```bash
docker system prune --all
```

---

## 10. Ejecutar contenedores

### Publicación de puertos

La sintaxis es:

```text
PUERTO_HOST:PUERTO_CONTENEDOR
```

Es decir, primero el puerto de fuera (host) y después el del contenedor. (lleva implicito que si no existe en local haga pull)

Ejemplo:

```bash
docker run -p 8080:80 -p 7080:7080 --name conBilling sotobotero/billingapp
```

Si no se especifica una etiqueta (`tag`), Docker utiliza `:latest` por defecto.

---

## 11. Iniciar contenedores

```bash
docker start NOMBRE_CONTENEDOR
```

---

## 12. Descargar imágenes de Docker Hub

```bash
docker pull nombre:version
```

---

## 13. Ejecutar contenedores en segundo plano

* `-d`: ejecuta el contenedor en background.
* `-e`: define variables de entorno.
* `-p`: publica puertos.

Ejemplo:

```bash
docker run \
  --ulimit memlock=-1:-1 \
  -d \
  --name postgres \
  -e POSTGRES_USER=sa \
  -e POSTGRES_PASSWORD=admin \
  -e POSTGRES_DB=product_db \
  -p 5432:5432 \
  postgres:13.3
```

---

## 14. Ver logs

```bash
docker logs NOMBRE_CONTENEDOR
```

---

## 15. Detener contenedores

```bash
docker stop NOMBRE_CONTENEDOR
```

---

## 16. Volver a iniciar un contenedor

```bash
docker start NOMBRE_CONTENEDOR
```

---

## 17. Reiniciar un contenedor

Si el contenedor ya está ejecutándose:

```bash
docker restart NOMBRE_CONTENEDOR
```

Equivale a realizar un `stop` seguido de un `start`.

DETENER todos: 
docker stop $(docker ps -q)

construir aparti de docker file (desde dde esta cd). Ekemplo: 
docker build -t billingapp --no-cache --build-arg JAR_FILE=target/*.jar .

el punto en doscker significa contexto

para subir a docher hub:

`priemro crear el tag;: 
docker tag billingapp imayordomo/udemy-billingapp opcional aqui :v1 por ejemplo
y luego
docker push imayordomo/udemy-billingapp:
e indicar etiqueta (sin nada latest)

pero primero logearse con 
docker login

parsa descargar repositorios privados hace flata. Y para salif
docker logout

para inicializaer el orquestador: (-f file)
docker compose -f stack-billing.yml up -d

para infromacion del contenedor: 
docker inspect NOMBRE

y en la parte network se peude ver que no tiene la misma red que otro container 

EMPLO DE Dockerfile de n backend: 
# Usa una imagen base con JRE 17 en Alpine
FROM eclipse-temurin:17-alpine

# Create a new group with specific uid and non-root user called "admin"
# Be sure that group id are not present on host. if already exist change by arbitrary other uid
RUN addgroup -g 1028 devopsc \
    && adduser -D -G devopsc admin

# Create a new mount point at /tmp on native host because a volume is more efficient and faster than filesystem
VOLUME /tmp

# Copiamos el jar a la imagen
ARG JAR_FILE
# Establece una variable de entorno para la contraseña de la base de datos
ARG DB_PASSWORD

# Establece la variable de entorno DB_PASSWORD con el valor del argumento
ENV DB_PASSWORD=$DB_PASSWORD

# Copia el archivo JAR
COPY ${JAR_FILE} /tmp/app.jar

# Change ownership of the /app directory to the "admin" user
RUN chown -R admin:devopsc /tmp

# Cambia al usuario 'admin'
USER admin

# Ejecutamos el jar al iniciar el contenedor
ENTRYPOINT ["java","-jar","/tmp/app.jar"]

Con uin orquestador puedo llamar a mas de ua a la ve y aplicar up, stop, rm, etc: 
docker compose -f stack-billing.yml rm

y se peude elimanar la chaceh de muchos sitios:

docker volume prune
docker system prune -all

Para ejecutar un script en un contenedro ( y en este ejemplo una termiinal en el backend con sh)

docker exec -it billingApp-back sh
y asi podemos ejecutar cosas como $echo $DB_PASSWORD para que te responda con la variable. 

sobre todo se hace para comprobar y revisar, no para crear cosas y demas. 

administrar recursos: 

docker stats

no se para que es esto: 
docker compose -f stack-billing.yml up -d --force-recreate

docker info: info de la instalacion de docker. 

orquestaciones docker swarm: 
docker stack deploy -c archivo.yml nombrequelevamosadar

y ver e estado de los servicios: 
docker service ls

y para uno concetro: 
docker stack ps NOMBRE 

para terminar servicios swarm: 
docker stack rm NOMBRE

para salir de docker swarm 
docker swarm leave --force 
