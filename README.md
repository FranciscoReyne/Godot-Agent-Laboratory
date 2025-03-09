# Godot Agent Laboratory

Godot Agent Laboratory es una adaptación de Agent Laboratory (https://github.com/SamuelSchmidgall/AgentLaboratory) para Godot. :D!!.

Para esto necesitaremos configurar un servidor Python local que corra en la misma máquina que Godot, y que este se comunique con él a través de WebSockets, HTTP (REST API) o incluso sockets TCP/UDP directos.

## Integración de Python en Godot 4.4:

Godot utiliza principalmente GDScript como su lenguaje de scripting nativo. Aunque existen esfuerzos para integrar Python en Godot, como el proyecto godot-python (https://pythonrepo.com/repo/touilleman-godot-python-data-validation), la integración no es completa ni oficial. Esto puede limitar la capacidad de ejecutar código Python directamente dentro de Godot.

## Integración con Ollama

Queremos usar **Ollama** para ejecutar modelos LLM localmente sin depender de OpenAI, lo que nos da más control sobre la privacidad y costos. Para ello, Godot se comunicará con un servidor Python que interactúe con Ollama y devuelva respuestas en tiempo real.

📌 **Opciones para la Comunicación**

### **REST API con FastAPI o Flask**
- **Ventaja**: Fácil de implementar y depurar.
- **Desventaja**: Puede ser más lento que WebSockets para comunicación en tiempo real.
- **Ejemplo**:
  - Godot envía datos con HTTPRequest a `http://127.0.0.1:8000/predict`.
  - El servidor Python devuelve una respuesta con los cálculos.

### **WebSockets con `websockets` en Python**
- **Ventaja**: Más rápido para comunicación en tiempo real.
- **Desventaja**: Requiere un poco más de código en Godot y Python.
- **Ejemplo**:
  - Godot establece una conexión WebSocket con el servidor Python en `ws://127.0.0.1:8765`.
  - El servidor recibe comandos y envía respuestas de inmediato.

### **Sockets TCP/UDP personalizados**
- **Ventaja**: Mayor flexibilidad y velocidad para grandes volúmenes de datos.
- **Desventaja**: Más trabajo de implementación.
- **Ejemplo**:
  - Se usa socket en Python y `StreamPeerTCP` en Godot.

## 📌 Arquitectura recomendada

### **Godot (Front-end y Simulación de Agentes)**
- Define los agentes, visualización y comportamiento básico.
- Se comunica con el servidor Python para obtener decisiones de IA.

### **Servidor Python (IA y Cálculo con Ollama)**
- Corre localmente en `127.0.0.1` y procesa datos.
- Usa Ollama para ejecutar modelos LLM.
- Devuelve respuestas a Godot en tiempo real.

Usaremos **WebSockets**, ya que son ideales para **comunicación en tiempo real**, lo que es clave si queremos que los agentes en Godot interactúen dinámicamente con la IA en Python.

---

## **📌 Pasos para Implementarlo**

### **1️⃣ Configurar un servidor WebSocket en Python**
- Usaremos la librería `websockets` para manejar la conexión.
- El servidor procesará los datos y enviará respuestas en tiempo real.

### **2️⃣ Configurar WebSockets en Godot**
- Usaremos la clase `WebSocketPeer` en GDScript.
- Godot se conectará al servidor y enviará/recibirá información.

---

## **📌 Recursos Necesarios**

### 🔹 **Python**
- `websockets` (`pip install websockets`)
- `asyncio` para manejar la comunicación de manera asíncrona
- `ollama` para manejar los modelos LLM

### 🔹 **Godot 4.4**
- `WebSocketPeer` en GDScript
- Scripts para manejar el envío/recepción de datos

### 🔹 **Opcionales** (si se usa IA avanzada):
- FastAPI (si en el futuro queremos una REST API junto con WebSockets)

---

## **📌 ¿Cómo Funcionará?**

1. **Godot inicia la conexión WebSocket** a `ws://127.0.0.1:8765`.
2. **El servidor Python recibe los datos, los procesa** (ejemplo: consulta a Ollama para generar respuestas).
3. **Python envía la respuesta a Godot**, que actualiza los agentes en tiempo real.

---

## **🎯 Implementación del Sistema**

### **1️⃣ Servidor WebSocket en Python**

📌 **Instala las librerías necesarias**:
```sh
pip install websockets asyncio ollama
```

📜 **Código en Python (`server.py`)**
```python
import asyncio
import websockets
import json
import ollama

async def handle_message(websocket, path):
    async for message in websocket:
        print(f"Mensaje recibido: {message}")
        
        data = json.loads(message)
        prompt = data.get("message", "Dime algo interesante")
        
        # Llamar a Ollama para generar respuesta
        response = ollama.chat(model="llama3", messages=[{"role": "user", "content": prompt}])
        
        # Enviar respuesta a Godot
        await websocket.send(json.dumps({"response": response["message"]["content"]}))

async def start_server():
    server = await websockets.serve(handle_message, "localhost", 8765)
    print("Servidor WebSocket iniciado en ws://localhost:8765")
    await server.wait_closed()

asyncio.run(start_server())
```

---

### **2️⃣ Cliente WebSocket en Godot (GDScript)**

📜 **Código en Godot (`websocket.gd`)**
```gdscript
extends Node

var websocket = WebSocketPeer.new()
var connected = false

func _ready():
    var err = websocket.connect_to_url("ws://localhost:8765")
    if err != OK:
        print("Error al conectar WebSocket: ", err)
    else:
        print("Intentando conectar al servidor...")

func _process(delta):
    websocket.poll()
    if websocket.get_ready_state() == WebSocketPeer.STATE_OPEN and not connected:
        connected = true
        print("Conectado al servidor WebSocket!")
        send_message("Hola Ollama!")
    elif websocket.get_ready_state() == WebSocketPeer.STATE_CLOSED and connected:
        connected = false
        print("Conexión cerrada")
    while websocket.get_available_packet_count() > 0:
        var message = websocket.get_packet().get_string_from_utf8()
        print("Respuesta desde Python: ", message)

func send_message(msg):
    var message = JSON.stringify({"message": msg})
    websocket.send_text(message)
```

---

🚀 Ahora puedes probarlo y expandirlo! 😃 🚀


 Buena suerte!!
