# Portón de Garaje Inteligente con ESPHome

## Descripción

Este proyecto utiliza un ESP8266 y ESPHome para integrar un portón de garaje con Home Assistant.

La solución permite:

- Abrir y cerrar el portón desde Home Assistant.
- Conocer el estado actual del portón (abierto o cerrado).
- Actualizar el firmware mediante OTA (Over The Air).
- Integrarse de forma nativa con Home Assistant mediante la API de ESPHome.

El sistema utiliza:

- Un relé conectado al pulsador del motor del portón.
- Un sensor magnético (reed switch) o sensor de contacto para detectar si el portón está abierto o cerrado.

---

## Características

- Integración nativa con Home Assistant.
- Control remoto del portón.
- Detección de estado abierto/cerrado.
- Actualizaciones OTA.
- Punto de acceso de emergencia (Fallback AP).
- Comunicación cifrada con Home Assistant.
- Recuperación automática después de reinicios.

---

## Hardware Requerido

### Microcontrolador

- ESP8266 compatible con ESPHome.
- Ejemplo: ESP-01, ESP-01S, ESP8266 Relay Board o similar.

### Relé

- Módulo relé de 5V o 3.3V compatible con ESP8266.

### Sensor de Estado

- Sensor magnético tipo Reed Switch.
- Sensor de contacto para puertas.
- Cualquier sensor digital que permita detectar abierto/cerrado.

### Fuente de Alimentación

- Fuente adecuada para alimentar el ESP8266 y el relé.

---

## Funcionamiento General

El sistema se basa en dos componentes principales:

### Sensor de Estado

Conectado al pin:

yaml GPIO16 

Su función es detectar si el portón se encuentra:

- Abierto
- Cerrado

### Relé

Conectado al pin:

yaml GPIO5 

Simula la pulsación del botón físico del motor del portón.

Cuando Home Assistant solicita abrir o cerrar el portón:

1. El relé se activa durante 1 segundo.
2. El relé se desactiva.
3. El motor interpreta la acción como una pulsación del botón físico.

---

## Configuración Inicial

### Configurar la API de Home Assistant

Reemplazar:

yaml api:   encryption:     key: "[*****REEMPLAZAR CON SU PROPIO API KEY*****]" 

por la clave generada para su instalación de Home Assistant.

---

### Configurar la contraseña OTA

Reemplazar:

yaml ota:   password: "[*****REEMPLAZAR CON SU PROPIA CONTRASEÑA*****]" 

por una contraseña segura.

---

### Configurar el Fallback Hotspot

Reemplazar:

yaml ap:   ssid: "Porton-Garaje Fallback Hotspot"   password: "[*****REEMPLAZAR CON SU PROPIA CONTRASEÑA*****]" 

por una contraseña segura.

---

### Configurar el WiFi

En el archivo secrets.yaml:

yaml wifi_ssid: MiRedWiFi wifi_password: MiContraseña 

---

## Sensor de Estado del Portón

### Configuración

yaml binary_sensor:   - platform: gpio     pin:       number: GPIO16       mode: INPUT_PULLDOWN       inverted: true 

### Función

Este sensor determina si el portón está abierto o cerrado.

La lógica implementada considera:

| Estado del Sensor | Estado del Portón |
|------------------|------------------|
| ON | Abierto |
| OFF | Cerrado |

---

## Relé de Control

### Configuración

yaml switch:   - platform: gpio     id: relay1     pin:       number: GPIO5 

### Características

yaml restore_mode: ALWAYS_OFF 

Esto garantiza que el relé permanezca apagado después de reinicios o pérdidas de energía.

---

## Entidad Cover

ESPHome expone el portón como una entidad de tipo:

yaml cover 

Lo que permite utilizar todas las funcionalidades estándar de Home Assistant para puertas de garaje.

### Estado

La entidad obtiene su estado a partir del sensor conectado al GPIO16.

cpp if (id(garage_door_contact).state) {   return COVER_OPEN; } else {   return COVER_CLOSED; } 

---

## Apertura del Portón

Cuando se ejecuta la acción de abrir:

yaml open_action:   - switch.turn_on: relay1   - delay: 1s   - switch.turn_off: relay1 

El relé se activa durante 1 segundo simulando la pulsación del botón físico.

---

## Cierre del Portón

Cuando se ejecuta la acción de cerrar:

yaml close_action:   - switch.turn_on: relay1   - delay: 1s   - switch.turn_off: relay1 

La lógica es idéntica a la apertura.

Esto es común en motores de portón donde un único pulsador controla ambas acciones.

---

## Consideraciones Importantes

### Motores con Pulsador Único

Esta configuración está diseñada para motores que funcionan mediante un único botón de control.

Normalmente el comportamiento es:

1. Pulsación → Abrir.
2. Pulsación → Detener.
3. Pulsación → Cerrar.

Verifique que su motor sea compatible con este esquema antes de utilizar la configuración.

---

### Sensor de Estado

La precisión del estado mostrado en Home Assistant depende completamente del sensor instalado.

Se recomienda:

- Utilizar un reed switch de buena calidad.
- Instalarlo firmemente.
- Verificar que detecte correctamente las posiciones abierta y cerrada.

---

### Seguridad

Antes de controlar el portón remotamente:

- Verifique que no existan obstáculos.
- Utilice contraseñas seguras.
- Limite el acceso al dashboard.
- Considere agregar confirmaciones o PIN en Home Assistant.

---

## Integración con Home Assistant

Una vez agregado a ESPHome, Home Assistant detectará automáticamente:

### Entidad Cover

text cover.garage_door 

### Sensor de Estado

text binary_sensor.garage_door_sensor 

Los nombres exactos pueden variar según la configuración del dispositivo.

---

## Flujo de Funcionamiento

text Home Assistant        │        ▼ Entidad Cover        │        ▼ ESPHome        │        ▼ GPIO5 → Relé        │        ▼ Motor del Portón  Estado del Portón        │        ▼ Sensor Magnético        │        ▼ GPIO16        │        ▼ ESPHome        │        ▼ Home Assistant 

---

## Posibles Mejoras Futuras

- Sensor adicional para detectar movimiento.
- Sensores independientes para abierto y cerrado.
- Notificaciones cuando el portón quede abierto.
- Cierre automático después de cierto tiempo.
- Integración con cámaras.
- Apertura basada en geolocalización.
- Control mediante RFID o teclado numérico.
- Registro de aperturas y cierres.

---

## Licencia

Uso libre para proyectos personales, educativos y de automatización residencial.

Puede modificarse y adaptarse según las necesidades de cada instalación.
