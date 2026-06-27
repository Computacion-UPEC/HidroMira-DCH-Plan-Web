Plan Operativo Normado: HidroMira-DCH-Plan-Web
1. Objetivo Principal
Sincronizar los datos de los sensores hidrometeorológicos a una plataforma web accesible desde internet, garantizando la disponibilidad de la información en tiempo real.

2. Premisa de Partida (Prueba de Concepto)
Este sistema ya ha sido aplicado y validado de forma exitosa con un solo sensor. El presente plan operativo toma esa experiencia previa como la línea base. El objetivo actual no es inventar el sistema, sino normar, documentar y preparar el entorno para replicar y escalar esta solución que ya funciona.

FASE 1: Detección del Entorno
El propósito de esta fase es mapear exhaustivamente cómo funciona el sistema actual (de 1 sensor) para poder replicarlo. Se divide en los siguientes entornos:

1.1. Entorno Físico y Hardware
Sensor Validado: Identificar exactamente las especificaciones del sensor que ya funciona (marca, modelo, tipo de señal analógica/digital).
Microcontrolador/Gateway: Documentar la placa o dispositivo que recibe los datos del sensor actual (ej. ESP32, Arduino, Raspberry Pi).
Infraestructura Física: Evaluar las condiciones del lugar donde estarán los nuevos sensores (energía eléctrica disponible, protección climática, distancia al gateway).
1.2. Entorno de Red y Conectividad
Topología Actual: ¿Cómo se conecta el sensor que ya funciona a internet? (WiFi local, cable Ethernet, red celular 4G/LoRa).
Capacidad de Red: Determinar cuántos sensores adicionales puede soportar la red actual sin perder paquetes de datos.
Seguridad IP: Mapear las direcciones IP, puertos abiertos y reglas de firewall que permitieron que el primer sensor enviara datos fuera de la red local.
1.3. Entorno de Datos y Protocolos (El "Idioma")
Protocolo de Comunicación: Normar cómo viaja la información. Si el primer sensor usa HTTP (enviando datos a una web), MQTT (publicando en un tema), o WebSocket, esto se debe estandarizar para los siguientes.
Formato del Payload: Documentar cómo llegan los datos (ej. formato JSON como {"sensor_id": "01", "valor": 15.5, "unidad": "cm"}).
1.4. Entorno de Servidor y Base de Datos (Backend)
Recepción: Identificar qué servidor está recibiendo los datos del primer sensor (un servidor propio, un servicio en la nube como AWS, ThingSpeak, etc.).
Almacenamiento: Detallar dónde se guardan los datos (MySQL, PostgreSQL, MongoDB, InfluxDB) y con qué frecuencia.
1.5. Entorno Web y Acceso Público (Frontend)
Plataforma Actual: Identificar la tecnología usada para mostrar los datos del primer sensor en internet (ej. página en PHP, dashboard en React, Node-RED, Grafana).
Accesibilidad: Verificar cómo accede el público a esta web (dominio, http/https, requisitos de login).
Estado de la Fase 1
 Inventario de hardware del sensor funcional completado.
 Diagrama de red de la prueba de concepto realizado.
 Protocolo de comunicación documentado.
 Servidor y base de datos validados.
