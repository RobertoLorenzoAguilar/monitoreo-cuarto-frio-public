# 🧊 IoT Cold Room Monitoring System con IA y Contenedores

Sistema inteligente de monitoreo ambiental para cámaras frigoríficas basado en la arquitectura **MAPE-K**, sensores IoT, un modelo LLM local y despliegue en contenedores Docker.

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://www.python.org/)
[![Node-RED](https://img.shields.io/badge/Node--RED-Enabled-red)](https://nodered.org/)
[![MQTT](https://img.shields.io/badge/MQTT-Broker-orange)](https://mqtt.org/)
[![MongoDB](https://img.shields.io/badge/Database-MongoDB-green)](https://www.mongodb.com/)
[![LLM](https://img.shields.io/badge/LLM-Local%20AI-informational)](https://huggingface.co/)
[![Docker](https://img.shields.io/badge/Containerized-Docker-blue)](https://www.docker.com/)
[![React](https://img.shields.io/badge/Frontend-React-61DAFB?logo=react)](https://reactjs.org/)
[![LangChain](https://img.shields.io/badge/Framework-LangChain-blueviolet)](https://www.langchain.com/)

---

## 📖 Descripción

Este sistema monitorea y gestiona cámaras frigoríficas en tiempo real utilizando una arquitectura distribuida con **7 contenedores Docker**. Captura datos ambientales (temperatura y humedad), detecta condiciones críticas, genera recomendaciones mediante un modelo de lenguaje local (LLM) y notifica a los usuarios vía Telegram. Los datos se visualizan en un dashboard interactivo basado en React.

> **Nota**: Este proyecto **en su mayoria no utiliza Docker Compose**. Los contenedores se gestionan manualmente con comandos `docker build`, `docker run`, etc.

---


## 📑 Tabla de Contenido

- [🧊 IoT Cold Room Monitoring System con IA y Contenedores](#-iot-cold-room-monitoring-system-con-ia-y-contenedores)
- [📖 Descripción](#-descripción)
- [🛠️ Características](#️-características)
- [🧠 Arquitectura MAPE-K](#-arquitectura-mape-k)
- [⚙️ Componentes](#️-componentes)
- [📦 Tecnologías](#-tecnologías)
- [🚀 Instalación y Configuración](#-instalación-y-configuración)
  - [Prerrequisitos](#prerrequisitos)
  - [1. Configurar Mosquitto (Broker MQTT)](#1-configurar-mosquitto-broker-mqtt)
  - [2. Configurar Node-RED](#2-configurar-node-red)
  - [3. Configurar MongoDB + Mongo Express](#3-configurar-mongodb--mongo-express)
  - [4. Configurar el Frontend (React)](#4-configurar-el-frontend-react)
  - [5. Configurar el Servicio MAPE-K](#5-configurar-el-servicio-mape-k)
  - [6. Configurar el Modelo LLM](#6-configurar-el-modelo-llm)
- [📈 Flujo de Datos](#-flujo-de-datos)
- [📸 Capturas](#-capturas)
- [📋 Tareas Pendientes](#-tareas-pendientes)
- [🛡️ Recomendaciones para Producción](#️-recomendaciones-para-producción)
- [📚 Referencias](#-referencias)

---

## 🛠️ Características

- 📡 Monitoreo en tiempo real vía MQTT.
- 🧠 Recomendaciones inteligentes con un modelo LLM local.
- 📊 Dashboard interactivo con histórico de datos.
- 🔔 Notificaciones automáticas vía Telegram para alertas críticas.
- 🧰 Despliegue contenerizado con Docker.
- 📁 Persistencia de datos en MongoDB (NoSQL).

---

## 🧠 Arquitectura MAPE-K

El sistema implementa el ciclo **MAPE-K** (Monitor, Analyze, Plan, Execute – Knowledge) para gestionar ambientes críticos:

1. **Monitor**: Captura datos de sensores simulados o físicos.
2. **Analyze**: Evalúa condiciones contra umbrales seguros.
3. **Plan**: Orquesta el Ejecutor de acciones.
4. **Execute**: Genera recomendaciones con el modelo LLM, Ejecuta acciones (notificaciones, ajustes).
5. **Knowledge**: Almacena datos históricos en MongoDB.

## Distribución Contenedores
<img width="1055" height="720" alt="468310515-3ee01b48-c806-493b-ab15-6cc4c0a5dcb0" src="https://github.com/user-attachments/assets/6337aea5-dce1-46fe-a684-7d5360eb8ca0" />

---

## ⚙️ Componentes

| Componente           | Función                                                                 |
|----------------------|------------------------------------------------------------------------|
| **Node-RED**         | Simula sensores y orquesta flujos MQTT.                                |
| **Mosquitto Broker** | Gestiona la comunicación MQTT entre sensores y procesadores.           |
| **MongoDB**          | Almacena datos históricos.                                            |
| **Mongo Express**    | Interfaz para visualizar datos de MongoDB.                            |
| **Dashboard React**  | Visualización en tiempo real con WebSockets.                           |
| **LLM Runner**       | Ejecuta el modelo de lenguaje para recomendaciones.                    |
| **Telegram Bot**     | Envía notificaciones en caso de valores fuera de rango.                |

---

## 📦 Tecnologías

- **Lenguajes**: Python 3.8+, JavaScript
- **Frontend**: React, WebSockets
- **Backend**: Node-RED, Python
- **Base de Datos**: MongoDB, Mongo Express
- **IA Local**: Modelo `ai/smollm2` (Hugging Face)
- **Broker MQTT**: Eclipse Mosquitto
- **Contenedores**: Docker

---

## 🚀 Instalación y Configuración

### Prerrequisitos

- Docker instalado.
- Node.js y npm para el frontend.
- Python 3.8+ para el servicio MAPE-K.
- Acceso a Telegram para configurar el bot.

### 1. Configurar Mosquitto (Broker MQTT)

```bash
# Crear contenedor
docker run -d --name mosquitto-container -p 1883:1883 -p 9001:9001 eclipse-mosquitto

# Acceder al contenedor
docker exec -it mosquitto-container /bin/sh

# Instalar nano
apk update && apk add nano

# Editar configuración
nano mosquitto/config/mosquitto.conf
```

Configura el archivo `mosquitto.conf`:
```
allow_anonymous true
listener 1883 0.0.0.0
protocol mqtt
listener 9001
protocol websockets
```

Reinicia el contenedor:
```bash
docker restart mosquitto-container
```

Habilita reinicio automático:
```bash
docker update --restart unless-stopped mosquitto-container
```

Prueba MQTT:
```bash
mosquitto_sub -h 192.168.16.76 -t "prueba/robert"
mosquitto_pub -h localhost -p 1883 -t "prueba/robert" -m "¡Hola MQTT desde Docker!"
```

### 2. Configurar Node-RED

```bash
# Crear contenedor
docker run -d -p 1880:1880 --name mynodered nodered/node-red
```

Accede a Node-RED en: `http://localhost:1880`

#### Generador de Datos Simulados (Node-RED)

Copia este código en un nodo `function` para simular datos de sensores:

```javascript
// Simula temperatura (0-8°C) y humedad (30-60%)
var minTempC = 0, maxTempC = 8;
var minHumidity = 30, maxHumidity = 60;
var randomTemperatureC = parseFloat((Math.random() * (maxTempC - minTempC) + minTempC).toFixed(2));
var randomHumidity = parseFloat((Math.random() * (maxHumidity - minHumidity) + minHumidity).toFixed(2));
var randomRSSI = -Math.floor(Math.random() * 0) - 40;
var randomInterval = Math.floor(Math.random() * (5 * 60 * 1000 - 1 * 60 * 1000 + 1)) + 1 * 60 * 1000;

var lastDate = context.historicalData && context.historicalData.length > 0
    ? new Date(context.historicalData[context.historicalData.length - 1].fecha_hora)
    : new Date();
var now = new Date(lastDate.getTime() + randomInterval);

var newEntry = {
    sensor: "Sensor_A1",
    fecha: now.toLocaleDateString(),
    hora: now.toLocaleTimeString(),
    temperatura_c: randomTemperatureC,
    temperatura_f: parseFloat((randomTemperatureC * 9 / 5 + 32).toFixed(2)),
    humedad: randomHumidity,
    rssi: randomRSSI,
    fecha_hora: now.toLocaleString(),
    ip: "192.168.1.133",
    ssid: "Robert-Wifi",
    mac: "48:E7:29:A6:0B:D4",
    firmware: "1.0.10",
    interval: 5000,
    name: "NCD-0BD4",
    timestamp: now.toLocaleString()
};

context.historicalData = context.historicalData || [];
context.historicalData.push(newEntry);
msg.payload = newEntry;
return msg;
```

Conecta a un nodo `inject` y un nodo `mqtt out`.

### 3. Configurar MongoDB + Mongo Express

Usa el repositorio: [cataniamatt/mongodb-docker](https://github.com/cataniamatt/mongodb-docker).

Credenciales sugeridas para MongoDB Compass:
- **Usuario**: admin
- **Contraseña**: pass

### 4. Configurar el Frontend (React)

En el directorio `./client`:

```bash
npm install
npm start
```

Accede al dashboard en: `http://localhost:3000`

### 5. Configurar el Servicio MAPE-K

Instala dependencias:
```bash
pip install -r requirements.txt
```

Ejecuta en modo prueba:
```bash
python planeador.py --interval 10 --log-file my_service.log
```

Ejecuta como daemon:
```bash
python planeador.py --interval 10 --log-file my_service.log --daemon
```

Detén el servicio:
```bash
kill $(cat /tmp/analyzer_service.pid)
```

> **Importante**: Oculta el token del bot de Telegram en el código de producción.

### 6. Configurar el Modelo LLM

Instala el plugin de modelos:
```bash
sudo apt-get update
sudo apt-get install docker-model-plugin
```

Prueba la instalación:
```bash
docker model version
docker model run ai/smollm2
```

Ejemplo de uso del LLM:
```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:12434/engines/v1", api_key="local-key")
response = client.completions.create(
    model="ai/smollm2:360M-Q4_K_M",
    prompt="¿Qué hacer si la temperatura es 2°C bajo lo esperado?",
    max_tokens=100,
)
print(response.choices[0].text)
```

---

## 📈 Flujo de Datos

1. Los sensores (físicos o simulados en Node-RED) envían datos vía MQTT.
2. Node-RED simula sensor por MQTT.
3. El sistema MAPE-K analiza los datos, detecta anomalías y almacena datos en MongoDB.
4. Si hay alertas:
   - Se envía una notificación vía Telegram.
   - El LLM genera recomendaciones.
5. El dashboard React muestra datos en tiempo real.

---

## 📸 Capturas

### Dashboard
<img width="1489" height="870" alt="468308388-a38aa4a7-8ebf-463f-8015-c9bfb29769db" src="https://github.com/user-attachments/assets/7d858e55-6ea5-4ef0-93b5-ad56b9a96108" />
<img width="1416" height="846" alt="468308440-609aa1cc-0dd8-41d8-86f8-df881c86df30" src="https://github.com/user-attachments/assets/d14d18bb-5edd-4694-980f-4bba3aef153f" />

### Recomendación LLM
<img width="656" height="945" alt="470943946-242df1e9-dcb2-4941-8435-7311be3a5da0" src="https://github.com/user-attachments/assets/3e667f3c-9a95-452f-b25d-8d554372a7d4" />

### Alertas Telegram
<img width="656" height="1086" alt="470944461-8b4358c5-26aa-4779-916e-241e97f81b38" src="https://github.com/user-attachments/assets/dfb62b93-d371-4b45-8cf4-6bf2dadccea3" />

### Simulación en Node-RED
<img width="1018" height="453" alt="469903985-7bd9634a-6161-4439-bf12-9caa231c3525" src="https://github.com/user-attachments/assets/74e5fde1-987e-48a4-b8c3-cb4ff648c3ec" />

### Paquetes recomendados para Node-RED
<img width="644" height="560" alt="470946977-586c59c7-475b-48e1-937a-c5a7fe6332a0" src="https://github.com/user-attachments/assets/b41386b6-e2f5-4d33-b5c8-8cdeabbf069a" />

---

## 📋 Tareas Pendientes

- [ ] Automatizar despliegue con `docker-compose`.
- [ ] Comentar el código según PEP 8.
- [ ] Añadir interfaz para configurar umbrales de alerta.
- [ ] Integrar actuadores físicos (ej. luces vía GPIO).
- [ ] Implementar autenticación en el dashboard.

---

## 🛡️ Recomendaciones para Producción

- Conectar sensores físicos.
- Desplegar el LLM en una máquina con GPU para mejor rendimiento.
- Habilitar encriptación en MQTT.
- Ocultar el token de Telegram en variables de entorno.

---

## 📚 Referencias

- [MongoDB + Mongo Express](https://github.com/cataniamatt/mongodb-docker)
- [Cliente React para IoT](https://github.com/jamalabdi2/IoT-Temperature-And-Humidity-Monitoring-System)
- [Modelos de IA Local con Docker](https://jggomezt.medium.com/building-local-ai-applications-integrating-docker-model-runner-genkit-and-langchain-d0dfb4a4dfa7)
