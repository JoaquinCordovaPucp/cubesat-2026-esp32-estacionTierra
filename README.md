# Estacion Tierra - ESP32 (PlatformIO)

Firmware de estacion terrena para un enlace LoRa con un CubeSat/vehiculo remoto, implementado en **ESP32** con **Arduino Framework** y **RadioLib**.

## Descripcion

Este proyecto escucha paquetes por LoRa, responde a pulsos de enlace (ACK), recibe telemetria binaria y la imprime en formato CSV por puerto serial.

Flujo principal actual:
- Inicia el modulo LoRa SX1276 en 915 MHz.
- Permanece en recepcion continua.
- Si recibe un paquete tipo `0`, responde con `ack`.
- Si recibe un paquete tipo `1`, confirma standby y sigue escuchando.
- Si recibe un paquete tipo `3`, interpreta telemetria binaria y la envia por serial en CSV.
- Al presionar un boton (GPIO 21 a GND), transmite `TomarDatosTotales`.

## Tecnologias

- ESP32 (`board = esp32dev`)
- Framework Arduino
- PlatformIO
- RadioLib (`jgromes/RadioLib@^7.6.0`)

## Requisitos

- VS Code con extension PlatformIO
- Una placa ESP32
- Modulo LoRa compatible SX1276
- Cable USB para programacion y monitoreo serial

## Configuracion del proyecto

Archivo `platformio.ini`:
- Entorno: `env:esp32dev`
- Plataforma: `espressif32`
- Frecuencia LoRa: `915.0 MHz` (configurada en codigo)
- `monitor_speed = 115200`

## Conexion de pines (segun el codigo)

Instancia del modulo:
- `SX1276 radio = new Module(5, 4, 22, 3);`

Interpretacion comun en RadioLib:
- NSS/CS -> GPIO 5
- DIO0 -> GPIO 4
- RST -> GPIO 22
- DIO1 -> GPIO 3

Boton:
- `BTN_PIN = 21`
- Conexion esperada: boton entre GPIO 21 y GND (usa `INPUT_PULLUP`).

## Protocolo de paquetes (resumen)

Se usa el primer byte como `TYPE`:
- `0`: pulso/enlace -> la estacion responde `ack`.
- `1`: estado standby.
- `3`: telemetria completa binaria (`TelemetryPacket`).

### Estructura `TelemetryPacket`

Campos principales:
- `TYPE`
- `VOLT` (mV)
- `INCX`, `INCY` (rad * 1000)
- `LON`, `LAT` (grados * 1e7)
- `TIME` (s * 10)
- `VVEL` (m/s * 10)
- `PRES` (Pa)
- `TEMP` (K * 100)
- `ECO2` (ppm)
- `UV` (UV * 100)
- `GYRX`, `GYRY`, `GYRZ` (rad/s * 1000)
- `ACCX`, `ACCY`, `ACCZ` (m/s^2 * 1000)
- `ALT` (m * 10)
- `CHK` (CRC-16)

## Compilar y cargar

Desde la raiz del proyecto:

```bash
pio run
pio run -t upload
pio device monitor -b 115200
```

## Salida serial

Cuando llega telemetria tipo `3`, se imprime una linea CSV con todos los campos del paquete en este orden:

`TYPE,VOLT,INCX,INCY,LON,LAT,TIME,VVEL,PRES,TEMP,ECO2,UV,GYRX,GYRY,GYRZ,ACCX,ACCY,ACCZ,ALT,CHK`

## Estado actual y pendientes

- Implementado: recepcion, ACK, parsing de telemetria tipo `3`, envio de comando por boton.
- Pendiente: manejo del `TYPE = 2` (marcado como TODO en `src/main.cpp`).
- Recomendado: validar longitud/CRC del paquete antes de parsear y registrar telemetria en SD si aplica.

## Estructura del repositorio

- `src/main.cpp`: logica principal de radio, boton e impresion de telemetria.
- `platformio.ini`: configuracion de compilacion y dependencias.
- `include/`, `lib/`, `test/`: carpetas base de PlatformIO.

## Licencia

Define aqui la licencia del proyecto (por ejemplo: MIT) si deseas publicarlo con terminos claros.
