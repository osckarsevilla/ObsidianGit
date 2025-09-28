---
telegraph_page_url: https://telegra.ph/Dallas-sonda-error-06-21
telegraph_page_path: Dallas-sonda-error-06-21
---
# El botoncito

Aquí está vuestro *BeackingChanges de cabezera*

En esta actualización tambien se cambio un componente relacionado con la sonda Dallas. El error es similar a esté
![[Error dallas .jpg]]


## Pasos a seguir
1- Como siempre os digo, lectura, lectura e intentar entender qué es lo que pone.

2- No darle al botoncito antes de saber qué ocurrirá, ¿Si nos funciona, porque vamos a tocar?

Después  de todo este rollo que os cuento, vamos al tema que te quema.

La solución es muy sencilla. Según no dicen en la [documentación](https://esphome.io/components/one_wire) de esta update el componente a cambiado y debemos declarar como tal. Os pongo el código para que simplemente copiar y cambiar el switch por el vuestro que tenéis declarado

```yaml important:3 fold title: "nueva configuración sonda dallas"
one_wire:
  - platform: gpio
    pin: GPIOXX. #cambiar por el switch que tengais declarado
```




## Atención, existe un error en esta update con la sonda Dallas. 
El error dice que no encuentra ningún device

![[Pasted image 20240622134315.png]]
[Aquí](https://github.com/esphome/esphome/pull/6860) lo tenéis que documentado. 
Declarar esto en vuestro archivo de configuración

```yaml fold
external_components:
    # https://github.com/esphome/esphome/pull/6860
  - source: github://pr#6860
    components: 
      - dallas_temp
      - gpio
      - one_wire
    refresh: 0s

one_wire:
  - platform: gpio
    pin: 4

# DS18B20 Temperature Sensor
sensor:
  - platform: dallas_temp
    resolution: 12
    name: "Temperature"
```


Y con esto se despide vuestro *BreackingChanges* de cabezera.

Recordad como dice @atareao la vida son dos días y uno acaba de pasar
