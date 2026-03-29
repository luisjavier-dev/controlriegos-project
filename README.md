# 💧 Sistema IoT de Control de Riego – Villamayor (2018)

Proyecto de automatización y monitorización remota de un sistema de riego basado en compuertas motorizadas, desarrollado en un entorno real con infraestructura limitada y control originalmente presencial y cobertura móvil justa.

---

## 🧭 Contexto

El sistema original requería supervisión manual continua por parte de un operario, con controles distribuidos en distintas zonas físicas separadas entre sí.

- Zona Pantano:
  - 3 compuertas motorizadas
  - Sensores de distancia Siemens Bero (1 sensor por compuerta)
  - Sonda de nivel de agua
  - Medición de nivel en la entrada de agua al pantano (acequia)
  - Sistema de control basado en autómata industrial Siemens S5

- Zona Enebro (a ~150m):
  - 1 compuerta motorizada
  - 1 sensor de distancia Siemens Bero

Esta separación física dificultaba la operación, obligando a desplazamientos constantes y ajustes manuales para mantener niveles adecuados de agua.

---

## 🎯 Objetivo del proyecto

Diseñar e implementar un sistema que permitiera:

- Control remoto de apertura y cierre de compuertas
- Monitorización en tiempo real de niveles de apertura y cierre de compuerta
- Monitorización en tiempo real de niveles de agua
- Acceso desde dispositivos móviles
- Reducción de intervención manual y desplazamientos

---

## 🚀 Solución implementada

Se desarrolló una solución IoT basada en:

- Nodos distribuidos (Raspberry Pi + Arduino)
- Comunicación mediante red móvil (3G) y enlaces inalámbricos
- Acceso remoto a través de servidor web y VPN
- Interfaces web para control y visualización en tiempo real

Además, se implementó un sistema de **regulación automática del nivel de agua**, capaz de ajustar dinámicamente la apertura de las compuertas de salida de agua en función de un valor objetivo definido por el operario.

---

## ⭐ Características principales

- Control remoto desde móvil
- Monitorización en tiempo real (actualización cada segundo)
- Sistema autoregulable de nivel de agua
- Arquitectura distribuida y robusta
- Integración de hardware y software en entorno real

---

A continuación se detalla la infraestructura técnica que hace posible el funcionamiento del sistema:


## 🛰️ Infraestructura y Conectividad

El sistema está diseñado para conectar dos zonas físicas independientes (**Zona Pantano** y **Zona Enebro**) mediante una arquitectura distribuida basada en VPN, permitiendo control remoto, monitorización en tiempo real y sincronización de datos.

### 🌐 Arquitectura de red

La comunicación se realiza a través de un servidor en la nube que actúa como punto central:

- Servidor virtual (hosting):
  - Servidor web para interfaces de control
  - Servidor VPN (OpenVPN)
  - Broker MQTT

Ambas zonas se conectan como clientes VPN, permitiendo acceso seguro desde cualquier ubicación.

---

### 📡 Nodo principal – Zona Pantano

Este nodo actúa como punto central del sistema:

- **Raspberry Pi 3**
  - Ejecuta la lógica del sistema en Python
  - Cliente OpenVPN
  - Publica datos al servidor MQTT y pasan a web
  - Gestiona la comunicación con Arduino vía USB

- **Conectividad**
  - Acceso a internet mediante **dongle USB 3G (Huawei + SIM)**
  - Enlace punto a punto con Zona Enebro mediante **Ubiquiti LiteBeam M5**

- **Arduino Uno**
  - Lectura de sensores ultrasónicos (nivel de agua)
  - Control de relés (motores de tajaderas)
  - Comunicación serie USB con Raspberry Pi

---

### 📡 Nodo secundario – Zona Enebro

Nodo remoto conectado al principal:

- **Raspberry Pi 3**
  - Cliente OpenVPN independiente
  - Envío de datos al servidor MQTT y pasan a web
  - Comunicación con Arduino

- **Conectividad**
  - Enlace inalámbrico directo con la Zona Pantano mediante Ubiquiti

- **Arduino Uno**
  - Lectura de sensor de nivel de acequia
  - Control de una tajadera motorizada mediante relés

---

### 🔗 Comunicación entre nodos

- Ambas Raspberry Pi funcionan como **clientes VPN independientes**
- La red permite:
  - Acceso remoto desde web
  - Intercambio de datos entre nodos
  - Supervisión centralizada

Además:

> La Raspberry Pi de la Zona Pantano actúa como nodo principal de salida a internet, facilitando la conectividad del sistema completo en un entorno sin infraestructura de red cableada.

---

### 🔒 Seguridad

- Comunicación cifrada mediante **OpenVPN**
- Acceso remoto restringido a la red privada virtual
- Sistema aislado de accesos externos directos
- SSID WiFi LAN oculta.
- Pantalla de login en la app con distintos permisos según roles.
  ![Pantalla Login](/images/Login.png)
  <img src="/images/Login.png" width="300">

---

### 🧩 Esquema de conectividad

A continuación se muestra el esquema completo de la infraestructura, incluyendo enlaces de red, tipo de conexión y dispositivos implicados:

![Esquema de red](/images/ConectividadRiegos.png)



## ⚙️ Funcionamiento del sistema

El sistema permite al operario (guarda) supervisar y controlar remotamente el estado de las tajaderas y los niveles de agua sin necesidad de desplazarse físicamente entre las distintas zonas.

A través de una interfaz web accesible desde el móvil, se puede visualizar y actuar sobre el sistema en tiempo real.

---

### 📱 Interfaz de control

Las interfaces web permiten:

- Visualizar el nivel del pantano
- Monitorizar la profundidad de la acequia
- Consultar la apertura de cada tajadera
- Actuar sobre la apertura y cierre de las compuertas

Las actualizaciones se realizan en tiempo real (≈1 segundo), proporcionando una supervisión continua del sistema.

#### Control de tajaderas

![Control Tajaderas](/images/.png)

#### Monitorización de niveles

![Niveles de agua](./img/niveles_iphone.png)

---

### 🤖 Regulación automática

Uno de los aspectos clave del sistema es su capacidad de autoregulación.

El operario puede establecer un nivel objetivo de agua en la acequia. A partir de este valor, el sistema ajusta automáticamente la apertura de las tajaderas:

- Si el nivel supera el objetivo → cierre progresivo
- Si el nivel es inferior → apertura progresiva

#### Ejemplo de funcionamiento:

- Nivel objetivo: **180 cm**
- Nivel actual: **185 cm** → cierre de 1 cm por ciclo
- Nivel actual: **175 cm** → apertura de 1 cm por ciclo

Este enfoque permite mantener un nivel estable sin necesidad de intervención constante.

---

### 🎯 Resultado

- Eliminación de desplazamientos continuos entre zonas
- Mayor precisión en el control del nivel de agua
- Supervisión remota en tiempo real
- Automatización parcial del sistema hidráulico
