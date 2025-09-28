---
title: "Guía: Obsidian con sincronización instantánea, gratuita y auto-hospedada"
source: "https://www.reddit.com/r/selfhosted/comments/1eo7knj/guide_obsidian_with_free_selfhosted_instant_sync/"
author:
  - "[[Timely_Anteater_9330]]"
published: 2024-08-09
created: 2025-09-25
description:
tags:
  - "clippings"
---
**TL;DR:** Llevo más de un mes usando Obsidian con el plugin [LiveSync](https://github.com/vrtmrz/obsidian-livesync) de [vrtmrz](https://github.com/vrtmrz) y, sin contar la pila de Arr, este plugin es, sin duda, el mejor servicio autoalojado que tengo en mi servidor. Lo uso varias veces al día y, a estas alturas, no puedo vivir sin él. Así que decidí devolverle algo a la comunidad, que tanto me ha enseñado, compartiendo mi experiencia y escribiendo también una guía detallada. Me di cuenta de que la mayoría de las guías pasan por alto pasos cruciales, pero, por otra parte, rara vez sé lo que hago, así que tómense mi guía con pinzas.

# La historia

Hace poco me embarqué en la búsqueda de un reemplazo para Apple Notes, que documenté [aquí](https://www.reddit.com/r/selfhosted/comments/1dnx38z/apple_notes_replacement/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) y buscaba algo que cumpliera con lo siguiente:

1. Poder autoalojarse en mi servidor Unraid.
2. Debe tener una aplicación para iOS, no algo a lo que se acceda en un navegador.
3. Sincronizar mis notas entre todos mis dispositivos de forma instantánea y sin problemas.

En este maravilloso sub-Reddit, Obsidian era constantemente recomendado. Así que descargué la aplicación para Windows 11 en mi escritorio y la aplicación para iOS en mi iPhone, y me quedé muy contento con lo pulida que estaba. No es de código abierto, pero estaba dispuesto a pasarlo por alto.

Luego me encontré con el obstáculo de sincronizar mis notas entre dispositivos, que Obsidian ofrece un servicio llamado Obsidian Sync por $4 al mes, pero quería autoalojar este aspecto, no quería depender de otra persona (preferencia personal). *Si no quieres autoalojar la sincronización, te recomiendo encarecidamente que apoyes a la empresa utilizando su servicio de sincronización.*

Me recomendaron un plugin para Obsidian llamado [LiveSync](https://github.com/vrtmrz/obsidian-livesync) de [vrtmrz](https://github.com/vrtmrz) que te permite autoalojar el proceso de sincronización. A continuación, una guía detallada sobre cómo configurarlo.

# Cómo funciona

Este "servicio" tiene 3 partes móviles. La aplicación Obsidian, el plugin LiveSync y la base de datos CouchDB en un contenedor docker. Aquí hay un desglose de cada uno:

1. **Aplicación Obsidian:** Instalas la aplicación en cada dispositivo. Yo la uso en un iPhone, un iPad, un portátil con Windows 10, un escritorio con Windows 11 y un cliente web (contenedor docker de [Linuxserver](https://docs.linuxserver.io/images/docker-obsidian/)). Cada dispositivo tiene una copia local de tus notas para que puedas seguir usándola sin conexión.
2. **CouchDB:** Aquí es donde se almacenará una copia de tus notas (el cifrado es una opción y también se recomienda).
3. **Plugin LiveSync:** El plugin es el que hace todo el trabajo pesado de sincronizar todos tus dispositivos. Esto se consigue conectándose a tu contenedor docker CouchDB autoalojado y almacenando una copia cifrada allí. Todos tus otros dispositivos se conectarán a la base de datos para obtener las notas actualizadas, lo que permite una sincronización instantánea.

# Docker Compose en Unraid

A continuación, el archivo docker compose solo para poner CouchDB en funcionamiento. Lo instalé en un servidor Unraid para que puedas editar las etiquetas y las variables de entorno para tu sistema operativo específico.

  couchdb-obsidian-livesync:
    container\_name: obsidian-livesync #nombre abreviado
    image: couchdb:3.3.3
    environment:
      - PUID=99
      - PGID=100
      - UMASK=0022
      - TZ=America/New\_York
      - COUCHDB\_USER=obsidian\_user # opcionalmente cámbialo
      - COUCHDB\_PASSWORD=password # definitivamente cámbialo
    volumes:
      - /mnt/user/appdata/couchdb-obsidian-livesync/data:/opt/couchdb/data
      - /mnt/user/appdata/couchdb-obsidian-livesync/etc/local.d:/opt/couchdb/etc/local.d
    ports:
      - "5984:5984"
    restart: unless-stopped
    labels:
      - net.unraid.docker.webui=http://\[IP\]:\[PORT:5984\]/\_utils # por alguna razón esto no funciona correctamente
      - net.unraid.docker.icon=https://couchdb.apache.org/image/couch@2x.png
      - net.unraid.docker.shell=bash

# CouchDB - Configuración inicial

1. Ve a la página de administración de CouchDB yendo aquí: [`http://192.168.1.0:5984/_utils`](http://192.168.1.0:5984/_utils) asegúrate de usar la dirección IP de tu servidor.
2. Inicia sesión con las credenciales que estableciste en el archivo Docker compose.
3. Haz clic en el icono `<->` en la parte superior izquierda, se expandirá el menú de iconos simples a iconos con texto, lo que facilitará seguir esta guía.
4. Haz clic en `Setup` en el menú de la izquierda.
5. Haz clic en `Configure as Single Node` e introduce las mismas credenciales del archivo Docker compose en los campos `Specify your Admin credentials` .
6. Deja todo lo demás igual y haz clic en `Configure Node`.

# CouchDB - Verificar la instalación

1. Vamos a verificar la instalación de CouchDB haciendo clic en `Verify` en el menú de la izquierda.
2. Haz clic en `Verify Installation` y, si todo va bien, debería aparecer un banner emergente que diga `Success! Your CouchDB installation is working. Time to Relax.` junto con 6 marcas de verificación junto a cada elemento de la tabla.

# CouchDB - Crear base de datos

1. Haz clic en el `Databases` en el menú de la izquierda.
2. Haz clic en `Create Database` en la parte superior derecha.
3. En `Database Name` introduce `obsidiandb`, o lo que quieras. Consejo: si tienes la intención de utilizar esta configuración para varios usuarios, cada usuario necesitará su propia base de datos, por lo que te recomiendo que nombres la base de datos para incluir el nombre del usuario, como: `obsidiandb_john` o `obsidiandb_jane` solo para que sea más fácil en el futuro.
4. En `Partitioning` selecciona `Non-partitioned - recommended for most workloads`. Una vez creada la base de datos, deberías ser redirigido a la página de configuración de la nueva base de datos. No tienes que hacer nada aquí.

# CouchDB - Configuración

1. Haz clic en `Configuration` en el menú principal de la izquierda. Las siguientes 9 entradas de configuración son lo que el script pretendía hacer automáticamente, pero yo quería hacerlo manualmente. Haz clic en `+ Add Option` en la parte superior derecha para cada entrada:
2. Sección: `chttpd` Nombre: `require_valid_user` Valor: `true`
3. Sección: `chttpd_auth` Nombre: `require_valid_user` Valor: `true`
4. Sección: `httpd` Nombre: `WWW-Authenticate` Valor: `Basic realm="couchdb"`
5. Sección: `httpd` Nombre: `enable_cors` Valor: `true`
6. Sección: `chttpd` Nombre: `enable_cors` Valor: `true`
7. Sección: `chttpd` Nombre: `max_http_request_size` Valor: `4294967296`
8. Sección: `couchdb` Nombre: `max_document_size` Valor: `50000000`
9. Sección: `cors` Nombre: `credentials` Valor: `true`
10. Sección: `cors` Nombre: `origins` Valor: `app://obsidian.md,capacitor://localhost,` [`http://localhost`](http://localhost/)

# Obsidian - Cliente Windows 11

1. Descarga e instala el cliente Obsidian para Windows 11 desde [aquí.](https://obsidian.md/download)
2. Una vez instalado, abre Obsidian.
3. Junto a `Create new vault` haz clic en el botón `Create` .
4. En el campo `Vault name` , ponle a tu Vault el nombre que quieras, yo simplemente le puse `Vault`. Puedes pensar en un vault como una "carpeta maestra" que contiene todas tus carpetas y notas. Algunos usuarios tienen diferentes vaults para diferentes aspectos de sus vidas, como `Work` o `Personal` , pero yo lo mantengo todo en un solo vault para facilitar su uso.
5. La siguiente configuración es `Location`, haz clic en `Browse`. Aquí es donde se guardará localmente tu vault. Creé una carpeta `Obsidian` en la carpeta `Documents` , pero puedes ponerla donde quieras.
6. Haz clic en `Create` y Obsidian debería abrirse en tu vault recién creado con 3 paneles de ventana. El siguiente paso es configurar el plugin LiveSync.

# Obsidian - Plugin LiveSync

1. Haz clic en el botón `options` (icono de piñón) en la zona inferior izquierda.
2. Haz clic en `Community plugins` y haz clic en el botón `Turn on community plugins` después de leer la divulgación de riesgos.
3. Junto a `Community plugins` haz clic en el botón `Browse` .
4. Busca `Self-hosted LiveSync`.
5. Solo debería aparecer 1 plugin y es el de `voratamoroz`, haz clic en él.
6. Haz clic en el botón `Install` y deja que se instale.
7. Haz clic en el botón `Enable` .
8. Haz clic en el botón `Open setting dialog` .
9. Haz clic en el botón `Options` .
10. En `Settings for Self-hosted LiveSync.` deberías ver una fila de 8 botones, haz clic en el 4º botón con el icono del satélite 🛰️.
11. Aquí es donde introduciremos los detalles de CouchDB autoalojado. Junto a `Remote Type` asegúrate de que `CouchDB` esté seleccionado en el menú desplegable.
12. En el campo `URI` escribe [`http://192.168.1.0:5984`](http://192.168.1.0:5984/) asegúrate de cambiarlo a la IP y el puerto de tu servidor.
13. En el campo `Username` escribe `osidian_user` o lo que hayas usado en el docker compose.
14. Lo mismo para el campo `Password` .
15. En el campo `Database name` escribe `obsidiandb` o como hayas llamado a tu base de datos antes en CouchDB.
16. Haz clic en el botón `Test` para probar la conexión a la base de datos CouchDB. Suponiendo que todo funcione correctamente, debería aparecer un texto emergente que diga `Connected to obsidiandb successfully`.
17. Haz clic en el botón `Check` para confirmar que la base de datos se configuró correctamente, debería haber una marca de verificación púrpura junto a cada elemento de la lista. Si no, debería haber un botón `Fix` junto al elemento en el que puedes hacer clic para que lo cree o lo corrija por ti, pero yo prefiero hacerlo manualmente.
18. Suponiendo que todo está bien hasta este punto, haz clic en el botón `Apply` junto a `Apply Settings`.
19. **Opcional pero recomendado:** desplázate hacia abajo hasta `End-to-end encryption` y actívalo y establece una frase de contraseña. Por favor, recuerda esta frase de contraseña, ya que todos tus otros dispositivos deben tener frases de contraseña coincidentes para poder descifrar tus notas. Haz clic en el botón rojo `Just apply`.
20. En el menú superior, en `Settings for Self-hosted LiveSync.` deberías ver una fila de 8 botones, haz clic en el 5º botón con el icono de actualización 🔄.
21. Junto a `Sync mode` selecciona `LiveSync` en el menú desplegable.
22. Puedes cerrar las ventanas `settings` , en la parte superior derecha de las notas deberías ver `Sync: zZz` , lo que significa que todo funciona correctamente y la sincronización está en modo de espera hasta que empieces a escribir algo.
23. Repite las instrucciones anteriores para todos los demás dispositivos.

# Proxy inverso

Recomiendo encarecidamente poner esto detrás de al menos un proxy inverso, yo uso Nginx Proxy Manager junto con Cloudflare Tunnels. Definitivamente lo necesitarás si planeas usar dispositivos móviles, ya que requieren HTTPS.

# Conclusión

Espero que esto te ponga en marcha. A medida que te familiarices con la aplicación, descubrirás lo genial que es Obsidian. Estaré encantado de responder a cualquier pregunta.

---

## Comments

> **SirSoggybottom** • [62 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/lhd3w6g/) •
> 
> De vez en cuando, aparece algo genial en este sub…
> 
> ¡Gracias por compartir! Nunca me molesté mucho con Obsidian y no tenía necesidad real de usarlo, pero ahora estoy jugando un poco con esta configuración.
> 
> Ojalá el desarrollador del plugin de sincronización en vivo hubiera elegido otra base de datos, para poder "automatizar" toda esta configuración para su despliegue. No me gusta tener stacks de Compose donde tengo que desplegar una parte, luego hacer pasos de configuración manual para continuar con el resto. Disfruto configurando lo máximo posible para un despliegue de "un solo clic". Con bases de datos como MariaDB, Postgres, etc., eso es fácilmente posible con scripts de inicialización para "pre-llenar" la base de datos, etc. Pero entiendo que CouchDB es un poco diferente, y estoy seguro de que el desarrollador tiene muy buenas razones para elegirlo sobre otras opciones, y yo no soy desarrollador, así que no tengo ni idea de los detalles. Es solo algo que me molesta de toda esta configuración, eso es todo. Pero una vez que está hecho, es un trabajo fantástico e increíble del desarrollador. *(Acabo de notar que el almacenamiento de objetos tipo S3 es compatible, aunque experimental, podría probar esa ruta para tener un stack completo sin pasos manuales)*
> 
> # No olvides darle una estrella en GitHub.
> 
> Una pequeña adición rápida:
> 
> No me gusta ejecutar **contenedores sin revisiones de estado**, especialmente cuando se trata de una base de datos (CouchDB en este caso). Afortunadamente, CouchDB proporciona un punto final de API adecuado para su estado, así que simplemente agregar esto al compose funciona:
> 
> healthcheck:
>   test: curl --fail -s http://localhost:5984/\_up | grep -Eo '\\"status\\":\\"ok\\"' || exit 1
>   start\_period: 60s
>   interval: 30s
>   timeout: 10s
>   retries: 3
> 
> *(Sé que esa expresión regular es horrible, pero odio las expresiones regulares con pasión y me di por vencido intentando aprenderlas, por favor no pierdas tu tiempo intentando explicármela, funciona, suficiente para mí…tos)*
> 
> Desafortunadamente, Linuxserver/KasmVNC no proporciona un punto final similar útil, pero al menos curl existe en su imagen, así que podemos verificar si el servidor web está activo y respondiendo correctamente o no:
> 
> healthcheck:
>   test: "curl --fail -s http://localhost:3000/ || exit 1"
>   start\_period: 60s
>   interval: 30s
>   timeout: 10s
>   retries: 3
> 
> *(Te sugiero modificar los valores específicos de ambas comprobaciones para que se ajusten a tu propia configuración y gusto, como el intervalo, etc.)*
> 
> Con esas dos comprobaciones agregadas en el compose, también puedes usar "depends\_on" con una condición para el estado saludable. Básicamente, el contenedor de Obsidian dependerá de que el contenedor de la base de datos esté en un estado saludable; de lo contrario, Obsidian se detendrá y esperará a que la base de datos vuelva a estar saludable. O al iniciar el stack completo, el contenedor de Obsidian esperará hasta que la base de datos esté realmente activa y lista, y solo entonces se iniciará. Esto probablemente aumentará el tiempo de inicio total, pero es la forma "correcta" de hacer este tipo de cosas para evitar problemas. Y, honestamente, pagar el precio de unos pocos segundos perdidos en el tiempo de inicio para un stack que luego funciona casi las 24 horas del día, los 7 días de la semana, y tener menos problemas, es un precio justo. Pero estoy seguro de que algunos estarán en desacuerdo, cada uno a su manera.
> 
> Puedes lograr esto simplemente agregando esta sección al servicio de Obsidian en el compose:
> 
> depends\_on:
>   obsidian-couchdb:
>     condition: service\_healthy
> 
> *(Reemplaza obsidian-couchdb con el nombre correcto del contenedor)*
> 
> track me
> 
> > **Timely\_Anteater\_9330** • [18 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/lhd4o5i/) •
> > 
> > Agradezco tus palabras amables y el increíble consejo.
> > 
> > También he batallado para entender las expresiones regulares… hasta el día de hoy no le entiendo. Eso junto con las automatizaciones YAML en Home Assistant, afortunadamente descubrí PyScript que me permite escribir automatizaciones en Python. Realmente me abrió nuevas posibilidades para automatizaciones complejas. Pero me desvié.
> > 
> > Volviendo a tu punto, uso UptimeKuma para monitorear mis contenedores docker junto con Dozzle para los logs, los cuales reviso un par de veces al mes para asegurarme de que no hay problemas ocultos. Agregar la verificación de estado es una gran capa extra para complementar esos dos. Gracias por esto, señor, tiene mi voto positivo.
> > 
> > **csobrinho** • [11 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/lrnqnvc/) •
> > 
> > Dale un vistazo a [https://regex101.com/](https://regex101.com/) , es un recurso genial para probar diferentes combinaciones y ver qué hace tu regex.

> **\[eliminado\]** • [0 points](https://reddit.com/) •
> 
> Personalmente, tuve muchísimos más problemas con obsidian-git por los conflictos de sincronización y la poca compatibilidad con móviles. Además, con livesync, podés estar escribiendo en cualquier dispositivo y se refleja al instante, sin problemas, en cualquier otro dispositivo conectado al servidor.
> 
> Es buena opción si buscas algo sencillo y gratis, aunque.
> 
> > **GhostV5** • [18 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/lhega2l/) •
> > 
> > Personalmente, tuve muchísimos más problemas con obsidian-git por los conflictos de sincronización y la poca compatibilidad con móviles. Además, con livesync, podés estar escribiendo en cualquier dispositivo y se refleja al instante, sin problemas, en cualquier otro dispositivo conectado al servidor.
> > 
> > Es buena opción si buscas algo sencillo y gratis, aunque.

> **Sqwrly** • [20 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/lhcaexz/) •
> 
> Usé el plugin un rato y tuve muchos problemas de sincronización. Me rendí y pagué Obsidian por un año.
> 
> > **Timely\_Anteater\_9330** • [11 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/lhcdgq3/) •
> > 
> > Interesante… indirectamente tocas un punto que debería mencionar; esto es un plugin que, en mi opinión, debería considerarse “beta” sin ninguna duda.
> > 
> > He tenido suerte de no encontrarme con ningún problema de sincronización todavía. Curioso, ¿hubo alguna acción en particular que provocara problemas de sincronización?

> **contagon** • [6 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/lhd06mg/) •
> 
> ¡Gracias por el artículo! A mí personalmente me fue mejor con el plugin [de guardado remoto](https://github.com/remotely-save/remotely-save)

> **AbstractDiocese** • [9 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/meyugmn/) •
> 
> Esto debería ser el modelo para cualquier tutorial de auto-hospedaje. Instrucciones claras, cronológicas y exhaustivas, con explicaciones de las elecciones y de qué debería pasar cuando haces esto o lo otro. Increíble de verdad. No creo haber configurado un servicio auto-hospedado tan rápido y fácil sin ningún problema. Una obra maestra de tutorial.

> **1WeekNotice** • [23 points](https://reddit.com/r/selfhosted/comments/1eo7knj/comment/lhbldke/) •
> 
> Gracias por este escrito tan increíble y claro.
> 
> Tenía una pregunta: ¿Cuál sería la diferencia/ventaja entre esto y usar algo como [syncthing](https://syncthing.net/) para mantener tus archivos sincronizados?
> 
> > Proxy Reverso
> 
> > Te recomiendo mucho poner esto detrás de, al menos, un proxy reverso. Yo uso Nginx Proxy Manager junto con Cloudflare Tunnels. Definitivamente lo vas a necesitar si piensas usar dispositivos móviles, ya que requieren HTTPS.
> 
> Además de esto, una mejor opción de seguridad sería usar una VPN como WireGuard. Puedes autohospedarla (mi preferencia personal, ya que tampoco me gusta depender de otros) o usar un servicio de terceros.
> 
> Esto reemplazaría a Cloudflare Tunnels y no sería accesible públicamente. Igual puedes usar un proxy reverso dentro de tu red para forzar la conexión HTTPS.
> 
> ¡Gracias otra vez por el escrito!