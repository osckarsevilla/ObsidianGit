---
telegraph_page_url: https://telegra.ph/Update-ESPHome-202460---19-Junio-2024-06-21
telegraph_page_path: Update-ESPHome-202460---19-Junio-2024-06-21
---


Como seguramente os salto la update de ESPHome, disteis al botoncito y no habéis leído que es lo que puede ocurrir o si se va a desatar la guerra mundial en tu casa.

Con esta update lo dispositivos que tienes con ESPHome no se actualizan, dando un error similar a este

  

![](https://telegra.ph/file/f987fba5d2b067ac3a5b6.png)

  

#### Pues aquí tenéis a vuestro BreeackingChanges de cabecera para qué os guie en este cambio y que sepáis qué hacer.

1 - Lectura, es importante antes de dar al botón, leer la info e intentar entender que cambios son, para saber si nos afecta a nosotros. Tenéis el enlace arriba y también antes de actualizar os lo ponen.

2 - Para esta actualización en concreto, se modifican 2 cosas

- 2.1 El componente [OTA](https://esphome.io/components/ota#esphome-ota-updates). Ahora está como componente, es decir, tienes que declararlo como cualquier otro, light, switch, sensor, etc y tiene su propia configuración.

  

```yaml
ota:
  - platform: esphome
    password: !secret ota_password #1

#1-cambia por tu passwd o aprovecha y lo llevas al archivo secrets
```

- 2.2 El componente [SAFE MODE](https://esphome.io/components/safe_mode.html#safe-mode). Habilita el modo seguro en caso de algún warning que provoque el inicio de la placa. Si después de 10 intentos no consigue levantar, este componente deshabilitará todo lo que tengas declarado menos wifi y las ota. De esta manera podrás recuperar tu dispositivo

```yaml
safe_mode:
```


Con estos sencillos pasos podrás tener otra vez todas tus placas ESPHome funcionando

  

Espero haberte ayudado.

#### Como dice @atareao La vida son dos días y uno ya paso