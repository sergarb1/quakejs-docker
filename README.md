<div align="center">
    
![logo](https://github.com/treyyoder/quakejs-docker/blob/master/quakejs-docker.png?raw=true)
# QuakeJS Docker (Versión Modificada – Runtime Patch)

![Docker Image CI](https://github.com/treyyoder/quakejs-docker/workflows/Docker%20Image%20CI/badge.svg)
</div>

### Un servidor de QuakeJS totalmente local y "dockerizado". Independiente y listo para la acción.

Esta guía utiliza la **imagen oficial existente**  
`treyyoder/quakejs:latest`  

sin necesidad de modificarla ni reconstruirla.

El funcionamiento correcto se consigue **sobrescribiendo el entrypoint en tiempo de ejecución**, para parchear dinámicamente el cliente web y lanzar el servidor Quake de forma totalmente local.

---

## 🚀 Cómo ejecutarlo con Docker Compose (Recomendado)

Ejemplo de `docker-compose.yml` usando directamente la imagen oficial y un **entrypoint inline**:

```yaml
services:
  quakejs:
    image: treyyoder/quakejs:latest
    container_name: quakejs
    environment:
      - HTTP_PORT=8088
      - SERVER=0.0.0.0
    ports:
      - "8088:80"
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
docker-compose up -d
```

---

## 🐳 Cómo ejecutarlo con Docker CLI (`docker run`)

No es necesario construir nada.
Docker descargará automáticamente la imagen si no existe localmente.

```bash
docker run -d \
  --name quakejs \
  -e HTTP_PORT=8088 \
  -e SERVER=0.0.0.0 \
  -p 8088:80 \
  -p 27960:27960 \
  --entrypoint sh \
  treyyoder/quakejs:latest \
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

Abre el navegador y comparte el enlace con tus amigos:

```
http://localhost:8088
```

Funciona también en red local usando la IP del host:

```
http://IP_DEL_SERVIDOR:8088
```

¡Que empiece el **fragging**! 🔫💥

---

## 🔧 Qué se modifica en tiempo de ejecución

* **Parcheo dinámico del cliente web (`index.html`)**

  * Se elimina el hostname fijo `quakejs`
  * Se usa automáticamente `window.location.hostname`
  * Se respeta el puerto definido por `HTTP_PORT`

* **Servidor 100 % local**

  * No depende de `content.quakejs.com`
  * Todo el contenido se sirve desde el propio contenedor

* **Un solo contenedor**

  * Apache sirve el cliente web
  * Node.js ejecuta el servidor Quake 3
  * Apache se lanza en background, Node queda como proceso principal

---

## ⚙️ server.cfg

El archivo `server.cfg` se carga automáticamente al iniciar el servidor.

Para personalizarlo, consulta la documentación oficial de Quake 3:

[https://www.quake3world.com/q3guide/servers.html](https://www.quake3world.com/q3guide/servers.html)

---

## 🧠 Notas

* No se modifica la imagen original
* No se requiere `Dockerfile`
* Ideal para:

  * LAN parties
  * Clases de redes / ASIR / FP
  * Laboratorios Docker
  * Servidores temporales

---

## 🙌 Créditos

* Imagen original: [https://github.com/treyyoder](https://github.com/treyyoder)
* QuakeJS fork: [https://github.com/begleysm](https://github.com/begleysm)
* Proyecto base: [https://github.com/inolen/quakejs](https://github.com/inolen/quakejs)
