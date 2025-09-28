---
telegraph_page_url: https://telegra.ph/LubeLogger-09-10-2
telegraph_page_path: LubeLogger-09-10-2
tags:
  - docker
  - "#writefreely"
  - "#telegraph"
---
# Lubelogger

### Mantenimiento de tu vehículo



Con este contenedor podrás llevar el control de tu vehículo. 
Repostajes, cambios de aceite, ITV, cambios de batería, etc.
En su [web](https://lubelogger.com/) tienes una [documentación](https://docs.lubelogger.com/) extensa y fácil de seguir. 

Tienes una demo para que pruebes y veas que se trata, es de agradecer esto, ya que algunas veces crees que te puede valer y después te llevas un chasco. En el repositorio de [GitHub](https://github.com/hargata/lubelog) tienes el docker-compose y más opciones, si quieres lanzarlo ya con **traefik** y **postgresql**. Te animo a que des un vistazo.

---
---

Aquí tienes el docker compose para que solo tengas que copiar y listo, bueno tienes que hacer unos pequeños cambios, pero nada difícil. Yo modifico algunas cosas, los volúmenes se crean directamente en la carpeta donde está el archivo docker-compose así está todo más claro

1- Si tienes el puerto 8080 ocupado con otro contenedor, solo cambia por el que tengas libre.
Ejemplo
```yaml
ports:
  - 8087:8080
```
2 - Añadir el idioma español a la interface es sencillo. Los pasos los tienes [aquí](https://docs.lubelogger.com/Translations). 



> Básicamente es,
   .  Descargar el archivo en formato .json 
   .  Moverte hasta **"Settings"**
   .  En  **"Manage Languages"** y subir el archivo que  descargaste

---
_**Aquí tienes ya el docker-compose**_ recuerda, creas una carpeta con el nombre de la aplicación en tu directorio Docker[^1] y dentro de esa carpeta el archivo docker-compose.yaml, dentro pegas este texto




---



 <!-- **Este el el archivo docker compose** -->

```yaml

services:
  app:
    image: ghcr.io/hargata/lubelogger:latest
    build: .
    restart: unless-stopped
    # volumes used to keep data persistent
    volumes:
      - ./config:/App/config
      - ./data:/App/data
      - ./translations:/App/wwwroot/translations
      - ./documents:/App/wwwroot/documents
      - ./images:/App/wwwroot/images
      - ./temp:/App/wwwroot/temp
      - ./log:/App/log
      - ./keys:/root/.aspnet/DataProtection-Keys
    ports:
      - 8080:8080

    environment:
      - TZ=Europe/Madrid
      - LANG=es_Es.UTF-8


```






[^1]: Por ejemplo, mi ruta es esta `~/Docker/Lubelogger`