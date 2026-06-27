# 💧 HidroMira-DCH-Plan-Web

### Plataforma Distribuida de Telemetría Hidrometeorológica e Industrial en Tiempo Real

> Sistema integral de adquisición, transmisión, persistencia y visualización analítica de variables hidrometeorológicas e industriales, con soporte para monitoreo predictivo basado en la norma ISO 20816-3.

[![Status](https://img.shields.io/badge/Estado-En%20Desarrollo%20Activo-orange)](#-estado-del-proyecto)
[![Architecture](https://img.shields.io/badge/Arquitectura-IoT%20Distribuida-0078D4)](#-arquitectura-del-sistema)
[![Server](https://img.shields.io/badge/Servidor-Linux%20(Ubuntu%2FDebian)-E95420)](#%EF%B8%8F-infraestructura-de-producción-linux)
[![Backend](https://img.shields.io/badge/Backend-PHP--FPM%20%7C%20Python%20%7C%20Flask-777BB4)](#-capa-de-backend-híbrido)
[![Database](https://img.shields.io/badge/Base%20de%20Datos-PostgreSQL%20%7C%20MySQL-336791)](#-capa-de-persistencia)
[![License](https://img.shields.io/badge/Licencia-MIT-green)](#-licencia)

---

## Tabla de Contenidos

1. [Resumen Ejecutivo](#-resumen-ejecutivo)
2. [Objetivos de Ingeniería](#-objetivos-de-ingeniería)
3. [Arquitectura del Sistema](#-arquitectura-del-sistema)
4. [Especificación Técnica de Componentes](#-especificación-técnica-de-componentes)
5. [Infraestructura de Producción (Linux)](#%EF%B8%8F-infraestructura-de-producción-linux)
6. [Normativa Técnica Aplicada](#-normativa-técnica-aplicada)
7. [Modelo de Seguridad](#-modelo-de-seguridad)
8. [Estructura del Repositorio](#-estructura-del-repositorio)
9. [Esquema de Base de Datos](#-esquema-de-base-de-datos)
10. [Stack Tecnológico](#-stack-tecnológico)
11. [Instalación y Despliegue](#-instalación-y-despliegue)
12. [API REST — Referencia de Endpoints](#-api-rest--referencia-de-endpoints)
13. [Casos de Uso](#-casos-de-uso)
14. [Estado del Proyecto](#-estado-del-proyecto)
15. [Equipo de Desarrollo](#-equipo-de-desarrollo)
16. [Licencia](#-licencia)

---

## 📖 Resumen Ejecutivo

**HidroMira-DCH-Plan-Web** es una plataforma de ingeniería de software y hardware desarrollada para la Central Hidroeléctrica Mira, orientada a la supervisión continua del comportamiento vibracional de equipos hidroeléctricos y la integración de variables hidrometeorológicas mediante sensores IoT, procesamiento de datos y visualización en tiempo real.

El sistema implementa una arquitectura distribuida desacoplada que garantiza alta disponibilidad, resiliencia a fallos de red y procesamiento concurrente, desplegada sobre servidores Linux de grado industrial con un backend híbrido PHP/Python.

### Principios de Diseño

| Principio | Descripción |
|:---|:---|
| **Modularidad** | Cada componente evoluciona de forma independiente sin afectar al resto del sistema. |
| **Escalabilidad Horizontal** | Incorporación de nuevos nodos de monitoreo sin rediseñar la plataforma. |
| **Interoperabilidad** | Compatibilidad con protocolos estándar de IoT (MQTT, HTTP/REST, ModBus RTU). |
| **Tolerancia a Fallos** | Conmutación automática a almacenamiento local ante caídas de la base de datos. |
| **Observabilidad** | Registro centralizado de eventos y diagnóstico de fallos en tiempo real. |
| **Mantenibilidad** | Estructura organizada, documentada y con separación de responsabilidades. |

---

## 🎯 Objetivos de Ingeniería

- Diseñar e implementar una arquitectura IoT modular y escalable para monitoreo hidrometeorológico.
- Centralizar la adquisición de datos provenientes de múltiples sensores y estaciones.
- Garantizar la transmisión confiable de información hacia la plataforma web.
- Clasificar automáticamente el estado operativo de la maquinaria según la norma **ISO 20816-3**.
- Facilitar la visualización en tiempo real mediante paneles interactivos con diagnóstico analítico.
- Implementar un sistema de alertas multicanal (correo electrónico, Telegram, MQTT, ThingSpeak).
- Proporcionar herramientas de mantenimiento predictivo basadas en correlación vibración-rendimiento.
- Estandarizar la incorporación de nuevos nodos de monitoreo.
- Mantener documentación técnica que facilite futuras ampliaciones.

---

## 🧩 Arquitectura del Sistema

El sistema sigue una arquitectura de capas desacopladas que permite el aislamiento de fallos y la escalabilidad independiente de cada componente:

```text
┌──────────────────────────────────────────────────────────────────────────┐
│                      CAPA DE ADQUISICIÓN (Edge)                         │
│                                                                          │
│  ┌────────────────┐    ┌──────────────────────────────────────────────┐  │
│  │ Sensores       │───>│ Nodos IoT (ESP32 / Arduino)                 │  │
│  │ • Vibración    │    │ • Filtrado de señal                         │  │
│  │ • Hidrológicos │    │ • Cálculo local de variables                │  │
│  │ • Meteorológ.  │    │ • Control de actuadores (Motor DC / Servo)  │  │
│  └────────────────┘    └──────────────────┬───────────────────────────┘  │
└───────────────────────────────────────────┼──────────────────────────────┘
                                            │
                      WiFi / Ethernet / LoRa / MQTT / HTTP POST (JSON)
                                            │
┌───────────────────────────────────────────┼──────────────────────────────┐
│              CAPA DE TRANSPORTE Y PROXY (Nginx)                         │
│                                                                          │
│  • Enrutamiento L7              • Rate Limiting (Anti-DoS)              │
│  • Terminación SSL/TLS          • Balanceo de Carga                     │
│  • WebSocket Proxy              • Compresión gzip                       │
└────────────────┬──────────────────────────┬──────────────────────────────┘
                 │                          │
    ┌────────────▼────────────┐  ┌──────────▼──────────────────┐
    │   API CORE (PHP-FPM)    │  │   ANALÍTICA (Python)         │
    │                         │  │                              │
    │ • Validación de tramas  │  │ • Cálculo RMS / Amplitud    │
    │ • Integridad de datos   │  │ • Detección de Anomalías    │
    │ • Enrutamiento REST     │  │ • Clasificación ISO 20816   │
    │ • Publicación IoT       │  │ • Correlación Vibración-RPM │
    └────────────┬────────────┘  └──────────┬──────────────────┘
                 │                          │
    ┌────────────▼──────────────────────────▼──────────────────┐
    │          CAPA DE PERSISTENCIA (Fault-Tolerant)            │
    │                                                           │
    │  ┌─────────────────────┐    ┌──────────────────────────┐ │
    │  │ PostgreSQL / MySQL  │<-->│ JSON File Store (Fallback)│ │
    │  │ (Base Primaria)     │    │ (Resiliencia Automática)  │ │
    │  └─────────────────────┘    └──────────────────────────┘ │
    └───────────────────────────────────────────────────────────┘
                 │
    ┌────────────▼────────────────────────────────────────┐
    │         CAPA DE NOTIFICACIONES (Multicanal)         │
    │                                                      │
    │  📧 Email SMTP (Gmail/Azure)    📡 ThingSpeak Cloud  │
    │  💬 Telegram Bot API            📨 MQTT Broker       │
    │  🔗 Webhook HTTP POST                                │
    └──────────────────────────────────────────────────────┘
```

---

## 🔧 Especificación Técnica de Componentes

### 4.1. Capa de Adquisición de Datos y Control de Actuadores

#### Sensores de Vibración

El sistema soporta dos modos de comunicación con el transductor piezoeléctrico **WTVB01-485** de WitMotion:

| Parámetro | ModBus RTU | Protocolo Binario Nativo |
|:---|:---|:---|
| **Implementación** | `monitor_realtime.py` / `app.py` | `web_hidro_pro.py` |
| **Puerto Serial** | Configurable (por defecto `COM8`) | Fijo (`COM3`) |
| **Librería** | `minimalmodbus` | Decodificación binaria directa |
| **Dirección** | `0x50` (80 decimal) | `0x50` (80 decimal) |
| **Registros** | `58` ($V_x$), `59` ($V_y$), `60` ($V_z$) | `0x3D` ($V_x$), `0x3E` ($V_y$), `0x3F` ($V_z$) |
| **Escalado** | Valor / Factor configurable (1.0, 10.0, 100.0) | Entero con signo de 16 bits / 100.0 |
| **Modelo de Hilos** | Síncrono (auto-refresco 500 ms) | Asíncrono (`threading.Thread`) |

#### Control de Actuadores (Arduino)

El firmware del microcontrolador (`arduino_control.ino`) recibe comandos seriales a **9600 baudios** y controla:

- **Motor DC** (Puente H L298N): Pines ENA (PWM, Pin 9), IN1 (Pin 8), IN2 (Pin 7).
  - Protocolo: `M<velocidad>\n` donde velocidad ∈ [-255, 255].
- **Servomotor**: Pin de señal PWM en Pin 11.
  - Protocolo: `S<ángulo>\n` donde ángulo ∈ [10, 100] grados.
- **Confirmación**: Retorna `ACK: Servo ajustado a <valor>` o `ACK: Motor ajustado a velocidad <valor>`.

### 4.2. Capa de Backend Híbrido

#### API REST en PHP (PHP-FPM)
- Recepción de tramas JSON desde nodos IoT mediante método `POST`.
- Validación de sintaxis, comprobación de procedencia e integridad antes de la persistencia.
- Ejecución sobre PHP-FPM para máxima eficiencia computacional y bajo consumo de memoria RAM bajo cargas concurrentes.
- Adaptadores nativos para PostgreSQL (`PDO`) y MySQL/MariaDB.

#### Motor Analítico en Python
- Paneles de operación construidos con **Streamlit** y gráficas interactivas con **Plotly**.
- API REST complementaria construida con **Flask** + **Flask-CORS**.
- Cálculo de métricas estadísticas: RMS, Amplitud, Pico a Pico, Desviación Estándar.
- Detección de anomalías periódicas mediante análisis de desviación típica ($\text{valor} > \bar{x} + k \cdot \sigma$, con $k = 2.0$).
- Correlación Vibración-RPM: $\text{RPM} = \min(\text{RMS} \times 2400,\; 1200)$.
- Modelo de degradación de rendimiento por zona ISO (Zona A: 95-100%, B: 80-95%, C: 60-80%, D: <60%).

### 4.3. Capa de Persistencia (Fault-Tolerant)

El adaptador de almacenamiento (`db.py`) implementa un mecanismo de **conmutación automática transparente**:

1. **Modo Primario** (`DB_ENABLED = True`): Todas las operaciones CRUD se ejecutan contra PostgreSQL/MySQL.
2. **Modo Fallback** (`DB_ENABLED = False` o fallo de conexión): El sistema persiste y lee de archivos JSON locales (`users.json`, `historical_data.json`, `maintenance_log.json`) de forma imperceptible para el usuario.
3. **Copias de Seguridad**: Motor de respaldos automáticos diarios (ejecutado en hilo asíncrono al iniciar la aplicación) y manuales bajo demanda. Genera snapshots de todos los archivos JSON y volcados SQL estructurados.

### 4.4. Capa de Notificaciones Multicanal

El orquestador de alertas (`iot_handler.py`) gestiona el envío asíncrono de telemetría y alarmas:

| Canal | Descripción | Frecuencia |
|:---|:---|:---|
| **ThingSpeak** | Almacenamiento en la nube (5 campos: $V_x$, $V_y$, $V_z$, RMS, Zona ISO) | Cada 30 lecturas (~15 s) |
| **MQTT** | Publicación en tiempo real al topic `hidromira/vibraciones` | Cada lectura |
| **Email SMTP** | Alertas HTML enriquecidas con tabla ISO incrustada y botones de acción contextuales | Por cambio de zona |
| **Telegram** | Mensajes formateados con detalles de ejes y acción recomendada | Por cambio de zona |
| **Webhook** | HTTP POST con payload JSON completo | Cada lectura |

Las alertas por correo electrónico incluyen botones de acción dinámicos que enlazan directamente a la interfaz web a través de la URL pública de ngrok:
- **Zona B/C**: Botón "Programar Parada Programada" → sección de mantenimiento.
- **Zona D**: Botón "Parada de Emergencia" → panel de control de actuadores.
- **Zona A (restauración)**: Botón "Ver Monitoreo en Tiempo Real" → dashboard principal.

---

## 🖥️ Infraestructura de Producción (Linux)

El entorno de producción está optimizado para servidores **Linux (Ubuntu Server LTS / Debian)**:

### 5.1. Servidor Web y Proxy Inverso (Nginx)

```nginx
# Ejemplo de configuración Nginx para HidroMira
upstream php_backend {
    server unix:/run/php/php-fpm.sock;
}

upstream streamlit_monitor {
    server 127.0.0.1:8503;
}

upstream streamlit_dashboard {
    server 127.0.0.1:8502;
}

server {
    listen 443 ssl http2;
    server_name hidromira.ejemplo.com;

    ssl_certificate     /etc/letsencrypt/live/hidromira.ejemplo.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/hidromira.ejemplo.com/privkey.pem;

    # API PHP
    location /api/ {
        fastcgi_pass php_backend;
        include fastcgi_params;
    }

    # Monitor en Tiempo Real
    location /monitor/ {
        proxy_pass http://streamlit_monitor/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # Dashboard de Análisis
    location /dashboard/ {
        proxy_pass http://streamlit_dashboard/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 5.2. Demonización de Servicios (Systemd)

```ini
[Unit]
Description=HidroMira - Servicio de Monitoreo en Tiempo Real
After=network.target postgresql.service

[Service]
Type=simple
User=hidromira
WorkingDirectory=/var/www/hidromira
ExecStart=/var/www/hidromira/venv/bin/streamlit run monitor_realtime.py \
    --server.port 8503 --server.address 127.0.0.1
Restart=on-failure
RestartSec=5s
StandardOutput=append:/var/log/hidromira/monitor.log
StandardError=append:/var/log/hidromira/monitor_error.log

[Install]
WantedBy=multi-user.target
```

### 5.3. Seguridad SSL/TLS
- Certificados emitidos por **Let's Encrypt** mediante renovación automatizada con `Certbot`.
- Forzado de redirección HTTP → HTTPS en todas las rutas.

### 5.4. Rotación de Registros (Logrotate)

```
/var/log/hidromira/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 hidromira hidromira
}
```

---

## 📏 Normativa Técnica Aplicada

### ISO 20816-3 — Grupo 1: Máquinas Grandes con Soporte Rígido

La clasificación del estado operativo se realiza según los umbrales de velocidad de vibración RMS establecidos por la norma:

| Zona | Rango (mm/s) | Estado | Rendimiento Estimado | Acción Recomendada |
|:---:|:---|:---|:---:|:---|
| **A** | 0 — 0.25 | ✅ Aceptable | 95 – 100 % | Operación normal. Monitoreo rutinario. |
| **B** | 0.25 — 0.50 | ⚠️ Vigilancia | 80 – 95 % | Aumentar frecuencia de monitoreo. Mantenimiento preventivo en 2-3 meses. |
| **C** | 0.50 — 0.75 | 🔴 Corrección | 60 – 80 % | Inspección detallada urgente. Mantenimiento correctivo inmediato. |
| **D** | > 0.75 | 🚫 Inaceptable | < 60 % | **Detener operación inmediatamente.** Reparación mayor necesaria. |

### Datos Técnicos de la Máquina Monitoreada

| Parámetro | Valor |
|:---|:---|
| **Fabricante** | DELTA - DELFINI & CIA., S.A. |
| **Tipo de Turbina** | Francis |
| **Diámetro del Rodete** | 465 mm |
| **Velocidad de Rotación Nominal** | 1200 RPM |
| **Caída Neta** | 77.6 m |
| **Caudal Máximo** | 1.77 m³/s |
| **Potencia Mecánica Máxima** | 1165.1 kW |
| **Eficiencia del Generador** | 96.5 % |
| **Potencia Eléctrica Generada** | 1124.3 kW |
| **Diseñador** | Riccardo Delfini |
| **Grupo Normativo** | ISO 20816-3 — Grupo 1 (Soporte Rígido) |

---

## 🔒 Modelo de Seguridad

### Autenticación y Control de Acceso

El sistema implementa autenticación local con control de acceso basado en roles (RBAC):

| Rol | Usuario por Defecto | Permisos |
|:---|:---|:---|
| **Administrador** | `admin` | Acceso total. Configuración de red, alertas, gestión de usuarios. |
| **Ingeniero en Jefe** | `jefe` | Registro de mantenimientos. Informes técnicos. |
| **Técnico de Planta** | `tecnico1` | Monitoreo y control manual de actuadores. |

### Mecanismos Criptográficos

- **Derivación de Claves**: PBKDF2 con algoritmo SHA-256 y 120,000 iteraciones.
- **Salt Criptográfica**: Token aleatorio único de 16 bytes por usuario (`secrets.token_hex(16)`).
- **Recuperación de Contraseña**: Generación de contraseña temporal aleatoria de 8 caracteres enviada al correo electrónico registrado mediante SMTP seguro (STARTTLS).

---

## 📂 Estructura del Repositorio

```text
HidroMira-DCH-Plan-Web/
│
├── README.md                       # Documentación técnica principal
├── PLAN_OPERATIVO.md               # Planificación de fases de desarrollo
├── requirements.txt                # Dependencias del entorno Python
├── pyproject.toml                  # Metadatos del proyecto
├── schema.sql                      # DDL de la base de datos relacional
├── .env.example                    # Plantilla de variables de entorno
├── .gitignore                      # Exclusiones del control de versiones
│
├── firmware/                       # ── Código para hardware de adquisición
│   ├── arduino_control/            #    Firmware AVR para actuadores (Motor DC + Servo)
│   │   └── arduino_control.ino     #    Sketch principal del controlador serial
│   └── esp32/                      #    Firmware para nodos ESP32/ESP8266
│
├── backend/                        # ── Lógica del servidor
│   ├── api/                        #    API REST de recepción rápida (PHP-FPM)
│   ├── api_rest.py                 #    API REST complementaria (Flask)
│   ├── auth.py                     #    Módulo de autenticación y criptografía
│   ├── db.py                       #    Adaptador de persistencia con fallback JSON
│   ├── iot_config.py               #    Configuración centralizada de servicios IoT
│   ├── iot_handler.py              #    Orquestador de alertas y notificaciones
│   ├── control_arduino.py          #    Servidor web de control de actuadores (Flask)
│   └── database/                   #    Scripts SQL y modelos de datos
│
├── frontend/                       # ── Capa de presentación
│   ├── app.py                      #    Panel principal de gestión y análisis (Streamlit)
│   ├── monitor_realtime.py         #    Dashboard de telemetría en tiempo real (Streamlit)
│   ├── web_hidro_pro.py            #    Panel multihilo para sensores WitMotion (Streamlit)
│   ├── dashboard/                  #    Vistas dinámicas del operador (PHP/HTML5/JS)
│   └── assets/                     #    Recursos estáticos (CSS, imágenes, fuentes)
│
├── scripts/                        # ── Utilidades de diagnóstico y pruebas
│   ├── diagnostico_sensor.py       #    Diagnóstico de comunicación con el sensor
│   ├── test_sensor_auto.py         #    Test automatizado de lectura de registros
│   ├── test_thingspeak.py          #    Verificación de conectividad con ThingSpeak
│   ├── test_email.py               #    Prueba de envío de correos SMTP
│   ├── generate_demo_data.py       #    Generador de datos de demostración
│   ├── reset_admin.py              #    Restablecimiento de contraseña de administrador
│   └── verificar_hardware.py       #    Verificación de puertos seriales y hardware
│
├── docs/                           # ── Documentación técnica ampliada
│   ├── INSTRUCCIONES.md            #    Guía de ejecución y flujo de datos
│   ├── GUIA_IOT.md                 #    Configuración de servicios IoT
│   ├── PRODUCCION.md               #    Requisitos para puesta en producción
│   ├── CREDENCIALES.md             #    Gestión de credenciales y accesos
│   ├── CONFIGURAR_GMAIL.md         #    Configuración de alertas por correo
│   └── EXPONER_CON_NGROK.md        #    Acceso remoto mediante túneles ngrok
│
└── backups/                        # ── Directorio de resguardo de datos
```

---

## 🗄️ Esquema de Base de Datos

```sql
-- Gestión de accesos y roles
CREATE TABLE IF NOT EXISTS users (
    id              SERIAL PRIMARY KEY,
    username        VARCHAR(100) UNIQUE NOT NULL,
    display_name    VARCHAR(100),
    role            VARCHAR(50) NOT NULL,       -- 'tecnico', 'admin', 'ingeniero_jefe'
    email           VARCHAR(150),
    salt            VARCHAR(100) NOT NULL,
    password_hash   VARCHAR(256) NOT NULL,
    active          BOOLEAN DEFAULT TRUE
);

-- Series temporales de variables físicas
CREATE TABLE IF NOT EXISTS historical_data (
    id      SERIAL PRIMARY KEY,
    vx      DOUBLE PRECISION NOT NULL,          -- Velocidad eje X (mm/s)
    vy      DOUBLE PRECISION NOT NULL,          -- Velocidad eje Y (mm/s)
    vz      DOUBLE PRECISION NOT NULL,          -- Velocidad eje Z (mm/s)
    rms     DOUBLE PRECISION NOT NULL,          -- Valor RMS máximo
    zona    VARCHAR(5) NOT NULL,                -- Zona ISO: 'A', 'B', 'C', 'D'
    ts      TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Bitácora de mantenimiento y control operativo
CREATE TABLE IF NOT EXISTS maintenance_log (
    id          SERIAL PRIMARY KEY,
    fecha       DATE NOT NULL,
    tipo        VARCHAR(50) NOT NULL,           -- 'Preventivo', 'Correctivo', 'Emergencia'
    descripcion TEXT,
    tecnico     VARCHAR(100) NOT NULL
);
```

---

## 🛠 Stack Tecnológico

### Hardware

| Componente | Función |
|:---|:---|
| ESP32 / ESP8266 | Nodos IoT de adquisición y transmisión |
| Arduino Uno/Nano | Control de actuadores (motor DC, servomotor) |
| Puente H L298N | Driver de potencia para motor de corriente continua |
| Sensor WTVB01-485 | Transductor piezoeléctrico triaxial de vibración |
| Sensores hidrometeorológicos | Medición de variables ambientales |

### Protocolos de Comunicación

| Protocolo | Uso |
|:---|:---|
| HTTP/REST (JSON) | Transmisión determinista de lecturas al backend |
| MQTT | Telemetría en tiempo real (baja latencia) |
| ModBus RTU | Lectura de registros del sensor de vibración |
| Serial UART (9600 baud) | Comunicación bidireccional con Arduino |
| SMTP (STARTTLS) | Envío de alertas por correo electrónico |

### Backend y Servidores

| Tecnología | Rol |
|:---|:---|
| **PHP** (PHP-FPM) | Recepción y validación de tramas IoT |
| **Python 3.x** | Motor analítico, cálculos estadísticos, alertas |
| **Streamlit** | Dashboards interactivos de operación |
| **Flask** | API REST complementaria y servidor de control |
| **Nginx** | Proxy inverso, terminación SSL, balanceo de carga |
| **Linux** (Ubuntu/Debian) | Sistema operativo del servidor de producción |
| **Systemd** | Gestión de ciclo de vida de servicios |

### Bases de Datos

| Motor | Uso |
|:---|:---|
| PostgreSQL | Base de datos primaria de producción |
| MySQL / MariaDB | Alternativa relacional compatible |
| JSON File Store | Almacenamiento de respaldo con fallback automático |

### Frontend

| Tecnología | Uso |
|:---|:---|
| HTML5 / CSS3 / JavaScript | Interfaces web del operador |
| Plotly.js | Gráficas interactivas de series temporales |
| Bootstrap | Maquetación responsiva |
| Chart.js | Visualizaciones complementarias |

### Dependencias Python

```
minimalmodbus          # Comunicación ModBus RTU con el sensor
pyserial               # Control de puertos seriales
streamlit              # Framework de dashboards interactivos
streamlit-autorefresh  # Auto-refresco periódico de la interfaz
plotly                 # Gráficas interactivas
numpy                  # Cálculos numéricos y estadísticos
flask                  # Microframework para API REST
flask-cors             # Soporte CORS para la API
paho-mqtt              # Cliente MQTT para telemetría IoT
requests               # Cliente HTTP para ThingSpeak y Webhooks
psycopg2-binary        # Driver PostgreSQL
```

---

## 🚀 Instalación y Despliegue

### Requisitos Previos

- Python 3.10 o superior
- PostgreSQL 14+ (opcional, el sistema funciona con JSON como fallback)
- PHP 8.1+ con PHP-FPM (para la API de recepción)
- Nginx (para producción)
- Servidor Linux (Ubuntu 22.04 LTS recomendado)

### Instalación Local (Desarrollo)

```bash
# 1. Clonar el repositorio
git clone https://github.com/Computacion-UPEC/HidroMira-DCH-Plan-Web.git
cd HidroMira-DCH-Plan-Web

# 2. Crear y activar entorno virtual
python -m venv venv
source venv/bin/activate          # Linux/macOS
.\venv\Scripts\Activate.ps1       # Windows (PowerShell)

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Configurar variables de entorno
cp .env.example .env
# Editar .env con las credenciales correspondientes

# 5. Ejecutar el monitor en tiempo real
streamlit run monitor_realtime.py --server.port 8503

# 6. En otra terminal, ejecutar el panel de análisis
streamlit run app.py --server.port 8502

# 7. (Opcional) Iniciar la API REST
python api_rest.py
```

### Despliegue en Producción (Linux)

```bash
# 1. Instalar dependencias del sistema
sudo apt update && sudo apt install -y python3-venv python3-pip \
    postgresql nginx php-fpm php-pgsql certbot python3-certbot-nginx

# 2. Crear usuario del sistema
sudo useradd -r -s /bin/false hidromira

# 3. Desplegar la aplicación
sudo mkdir -p /var/www/hidromira
sudo cp -r . /var/www/hidromira/
cd /var/www/hidromira
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 4. Configurar base de datos PostgreSQL
sudo -u postgres createdb hidromira
sudo -u postgres psql -d hidromira -f schema.sql

# 5. Habilitar servicios systemd
sudo cp docs/hidromira-monitor.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now hidromira-monitor

# 6. Configurar Nginx y SSL
sudo cp docs/nginx-hidromira.conf /etc/nginx/sites-available/hidromira
sudo ln -s /etc/nginx/sites-available/hidromira /etc/nginx/sites-enabled/
sudo certbot --nginx -d hidromira.ejemplo.com
sudo systemctl reload nginx
```

---

## 📡 API REST — Referencia de Endpoints

Base URL: `http://<servidor>:5000`

| Método | Endpoint | Descripción | Parámetros |
|:---:|:---|:---|:---|
| `GET` | `/` | Información general de la API | — |
| `GET` | `/status` | Última lectura y estado actual del sistema | — |
| `GET` | `/datos` | Últimas N lecturas del histórico | `?limit=100` |
| `GET` | `/datos/rango` | Lecturas filtradas por rango de fechas | `?desde=ISO8601&hasta=ISO8601` |
| `GET` | `/estadisticas` | Estadísticas generales (promedios, máximos, distribución por zonas) | — |
| `GET` | `/alertas` | Últimas 50 lecturas en Zona B, C o D | — |

### Ejemplo de Respuesta (`/estadisticas`)

```json
{
  "total_lecturas": 12500,
  "periodo": {
    "inicio": "2026-01-07T10:00:00Z",
    "fin": "2026-06-27T16:00:00Z"
  },
  "estadisticas": {
    "vx": { "promedio": 0.15, "max": 0.35, "min": 0.05, "std": 0.08 },
    "vy": { "promedio": 0.12, "max": 0.28, "min": 0.03, "std": 0.06 },
    "vz": { "promedio": 0.09, "max": 0.22, "min": 0.02, "std": 0.05 }
  },
  "distribucion_zonas": { "A": 12000, "B": 400, "C": 80, "D": 20 }
}
```

---

## 📊 Casos de Uso

La plataforma puede adaptarse a los siguientes escenarios:

- Monitoreo de vibraciones en hidroturbinas y equipos rotativos.
- Estaciones hidrometeorológicas distribuidas.
- Control de calidad del agua en reservorios.
- Monitoreo de caudales y niveles hidrológicos.
- Seguimiento de variables ambientales (temperatura, humedad, presión).
- Sistemas de alerta temprana ante condiciones anómalas.
- Mantenimiento predictivo basado en análisis de tendencias.
- Proyectos de investigación académica en ingeniería.

---

## 🚀 Estado del Proyecto

El proyecto se encuentra en evolución activa a partir de una **Prueba de Concepto (PoC)** previamente validada, donde se comprobó exitosamente la integración entre un nodo sensor y una plataforma web.

La fase actual consiste en convertir dicha implementación en una arquitectura completamente documentada, modular y preparada para ambientes con múltiples dispositivos.

---

## 👥 Equipo de Desarrollo

Proyecto desarrollado por estudiantes e investigadores de la **Carrera de Computación** de la **Universidad Politécnica Estatal del Carchi (UPEC)**.

---

## 📄 Licencia

Este proyecto se distribuye bajo los términos de la licencia **MIT**. Consulte el archivo `LICENSE` para más información.

---

> **HidroMira-DCH-Plan-Web** proporciona una base sólida de ingeniería para el desarrollo de plataformas IoT orientadas al monitoreo hidrometeorológico e industrial, priorizando una arquitectura modular, escalable, resiliente y preparada para futuras integraciones en entornos de producción.
