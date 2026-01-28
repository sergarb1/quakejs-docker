   
![logo](https://github.com/treyyoder/quakejs-docker/blob/master/quakejs-docker.png?raw=true)
# QuakeJS Docker (Versión Modificada)

![Docker Image CI](https://github.com/treyyoder/quakejs-docker/workflows/Docker%20Image%20CI/badge.svg)
</div>

### Un servidor de QuakeJS totalmente local y "dockerizado". Independiente y listo para la acción.

Esta es una versión modificada para asegurar que el juego funcione correctamente (“para que vaya”) sin depender de servicios externos.  
Se han realizado ajustes para **parchear dinámicamente el cliente web (`index.html`) en tiempo de ejecución**, gestionando correctamente el hostname y el puerto HTTP.

---

## 🚀 Cómo ejecutarlo con Docker Compose (Recomendado)

Ejemplo de `docker-compose.yml` usando un **entrypoint inline**:

```yaml
services:
  quakejs:
    build: .
    container_name: quakejs
    environment:
      - HTTP_PORT=8080
      - SERVER=0.0.0.0
    ports:
      - "8080:80"
      - "27960:27960"
    entrypoint: >
      sh -c "
      cd /var/www/html &&
      sed -i \"s/'quakejs:/window.location.hostname + ':/g\" index.html &&
      sed -i \"s/':80'/':${HTTP_PORT}'/g\" index.html &&
      /etc/init.d/apache2 start &&
      cd /quakejs &&
      node build/ioq3ded.js
        +set fs_game baseq3
        +set dedicated 1
        +set fs_cdn 127.0.0.1
        +exec server.cfg
      "
````

Lanzar el servidor:

```bash
docker-compose up -d --build
```

---

## 🐳 Cómo ejecutarlo con Docker CLI (`docker run`)

Primero, construye la imagen localmente:

```bash
docker build -t quakejs-local .
```

Luego ejecútala sobrescribiendo el `entrypoint`:

```bash
docker run -d \
  --name quakejs \
  -e HTTP_PORT=8080 \
  -e SERVER=0.0.0.0 \
  -p 8080:80 \
  -p 27960:27960 \
  --entrypoint sh \
  quakejs-local \
  -c "
  cd /var/www/html &&
  sed -i \"s/'quakejs:/window.location.hostname + ':/g\" index.html &&
  sed -i \"s/':80'/':${HTTP_PORT}'/g\" index.html &&
  /etc/init.d/apache2 start &&
  cd /quakejs &&
  node build/ioq3ded.js
    +set fs_game baseq3
    +set dedicated 1
    +set fs_cdn 127.0.0.1
    +exec server.cfg
  "
```

---

## 🎮 Cómo jugar

Envía a tus amigos el enlace, por ejemplo:

```
http://localhost:8080
```

¡Y que empiece el **fragging**! 🔫💥

---

## 🔧 Cambios realizados para que funcione

* **Parcheo dinámico del cliente web**
  Se modifica `index.html` en tiempo de ejecución para:

  * Usar automáticamente el hostname del navegador
  * Respetar el puerto HTTP configurado (`HTTP_PORT`)

* **Servidor totalmente local**
  No depende de `content.quakejs.com`.
  Todo el contenido se sirve desde el propio contenedor.

* **Apache + servidor Quake en el mismo contenedor**
  Apache se lanza en segundo plano y el proceso principal es el servidor Node.js.

---

## ⚙️ server.cfg

Consulta la documentación de Quake 3 para configurar el servidor:

[https://www.quake3world.com/q3guide/servers.html](https://www.quake3world.com/q3guide/servers.html)

---

## 🙌 Créditos

Gracias a:

* [https://github.com/treyyoder](https://github.com/treyyoder) por la base original
* [https://github.com/begleysm](https://github.com/begleysm) por el fork de
  [https://github.com/inolen/quakejs](https://github.com/inolen/quakejs), del cual se deriva este proyecto