# 💧 Sistema IoT de Control de Riego – Villamayor (2018)

Proyecto de automatización y monitorización remota de un sistema de riego basado en compuertas motorizadas, desarrollado en un entorno real con infraestructura limitada y control originalmente presencial y cobertura móvil escasa.
Toda la funcionalidad que aquí se detalla, fue la que el Sindicato de riegos de Villamayor exigía. Tanto desde un principio, como con el paso del tiempo y mientras se desarrollaba el sistema, se fueron incrementando sus peticiones.
Sistema hecho totalmente a la medida de sus necesidades.

---

## 🧭 Contexto

El sistema original requería supervisión manual continua por parte de un operario, con controles distribuidos en distintas zonas físicas separadas entre sí. Armarios eléctricos con selectores de maniobra (subir / bajar compuertas)

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
  - Ejecuta la lógica del sistema en Python y transmite por USB a Arduino los movimientos de motores.
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
  - Enlace inalámbrico directo con la Zona Pantano mediante antena Ubiquiti

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
<p align="center">
  <img src="/images/1-Login.png" width="20%">
</p>
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
<p align="center">
  <img src="/images/5-EstadoTajaderas1.png" width="20%">
  <img src="/images/6-EstadoTajaderas2.png" width="20%">
</p>

- Actuar sobre la apertura y cierre de las compuertas
Las actualizaciones se realizan en tiempo real (≈1 segundo), proporcionando una supervisión continua del sistema.

#### Control de tajaderas
- El guarda responsable abre esta interfaz web, teclea los cm de apertura y pulsa "Establecer". En ese momento, la lógica programada en python, detecta que la altura objetivo es mayor que la altura real y envía al Arduino la orden de subir la compuerta activando el motor correspondiente y el sentido de giro para que levante.
Cuando detecte que la altura real sea igual o mayor que la objetivo, el motor para.
<p align="center">
  <img src="/images/9-Compuerta.png" width="20%">
</p>
Misma operativa y lógica para bajar compuerta.
Para cerrar del todo, simplemente se pulsa el boton web "Cerrar Totalmente"

#### Monitorización de niveles y gráfico informativo
- Se registra en base de datos la profundidad de la acequia de entrada al pantano cada hora.
<p align="center">
  <img src="/images/7-NivelAcequia.png" width="20%">
  <img src="/images/7.1-Grafico.jpg" width="20%">
</p>

#### Programación horaria de apertura
Permite programar como máximo hasta dos días furuos la apertura y cierre de las compuertas del panatano.
<p align="center">
  <img src="/images/3-ProgramacionHoraria.png" width="20%">
</p>

#### Histórico de movimientos en Base de Datos MySQL
Todos los movimiento se guardan en base de datos, pero la interfaz solo muestra los últimos movimientos
<p align="center">
  <img src="/images/11-HistorialOperaciones.png" width="20%">
</p>
---

### 🤖 Regulación automática

Uno de los aspectos clave del sistema es su capacidad de autoregulación.

El operario puede establecer un nivel objetivo de agua en la acequia de salida. A partir de este valor, el sistema ajusta automáticamente la apertura de las tajaderas:

- Si el nivel supera el objetivo → cierre progresivo
- Si el nivel es inferior → apertura progresiva
<p align="center">
  <img src="/images/8-Almenara1.jpg" width="20%">
</p>

#### Ejemplo de funcionamiento:

- Nivel objetivo: **180 cm**
- Nivel actual igual o mayor a: **185 cm** → cierre de 1 cm por ciclo
- Nivel actual igual o menor a: **175 cm** → apertura de 1 cm por ciclo

Este enfoque permite mantener un nivel estable sin necesidad de intervención constante.

---

### 🎯 Resultado

- Eliminación de desplazamientos continuos entre zonas
- Mayor precisión en el control del nivel de agua
- Supervisión remota en tiempo real
- Automatización parcial del sistema hidráulico

---

## 🧰 Tecnologías utilizadas

- **Python** (lógica de control en Raspberry Pi)
- **Arduino (C/C++)** (lectura de sensores y control de actuadores)
- **Raspberry Pi** (nodos de procesamiento)
- **OpenVPN** (conectividad segura remota creación de certificados)
- **HTML / CSS / JavaScript** (interfaces web de control)
- **PHP** Como apoyo a funciones Javascript

- **MySQL**:
  - Gestión de usuarios y permisos
  - Registro de niveles de agua
  - Histórico de operaciones (auditoría de acciones)
  
- **Redes**:
  - Conectividad 3G (dongle USB Huawei)
  - Enlaces inalámbricos punto a punto (Ubiquiti LiteBeam)


  ## 🛠️ Operación y soporte

El sistema fue administrado de forma remota, permitiendo su mantenimiento y supervisión sin necesidad de desplazamiento físico.

Como administrador del sistema, disponía de acceso a la red mediante VPN, pudiendo conectarme en cualquier momento a los nodos mediante SSH desde casa o desde el móvil.

---

### 🔧 Gestión remota

- Acceso a las Raspberry Pi mediante SSH
- Supervisión del estado de los servicios
- Diagnóstico de fallos en tiempo real
- Reinicio de servicios tras incidencias

La lógica del sistema estaba implementada como servicios que se iniciaban automáticamente al arrancar las Raspberry Pi, garantizando la continuidad del sistema tras reinicios.

---

### ⚠️ Incidencias y mejoras

Durante la operación del sistema se detectaron diversos fallos derivados principalmente de la falta de validación de datos en las primeras versiones.

Un caso relevante fue:

- Las lecturas recibidas (vía MQTT) debían ser valores numéricos enteros
- En ocasiones, los mensajes llegaban con caracteres erróneos
- Esto provocaba excepciones en el código Python y la caída del servicio

#### 🧠 Solución implementada

Se añadió una validación de entrada:

- Si el dato recibido es un número válido → se procesa
- Si no es válido → se descarta y se espera al siguiente mensaje

---

### 📈 Resultado

Tras implementar esta mejora:

- Se redujeron significativamente las caídas del sistema
- Disminuyeron las incidencias reportadas por el operario
- Se incrementó la estabilidad general del sistema

---

### 🧩 Consideraciones

El sistema fue desarrollado en un contexto sin herramientas avanzadas de monitorización o automatización actuales, lo que implicó una gestión manual de incidencias y optimización progresiva del código.
