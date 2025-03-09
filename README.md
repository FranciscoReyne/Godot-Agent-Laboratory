# Godot Agent Laboratory
Godot Agent Laboratory es una adaptación de Agent Laboratory (https://github.com/SamuelSchmidgall/AgentLaboratory) para Godot. :D!!. 

Para esto necesitaremos configurar un servidor Python local que corra en la misma máquina que Godot, y que este se comunique con él a través de WebSockets, HTTP (REST API) o incluso sockets TCP/UDP directos.

## Integración de Python en Godot 4.4:

Godot utiliza principalmente GDScript como su lenguaje de scripting nativo. Aunque existen esfuerzos para integrar Python en Godot, como el proyecto godot-python (https://pythonrepo.com/repo/touilleman-godot-python-python-data-validation), la integración no es completa ni oficial. Esto puede limitar la capacidad de ejecutar código Python directamente dentro de Godot.


📌 Opciones para la Comunicación

REST API con FastAPI o Flask

Ventaja: Fácil de implementar y depurar.
Desventaja: Puede ser más lento que WebSockets para comunicación en tiempo real.
Ejemplo:
Godot envía datos con HTTPRequest a http://127.0.0.1:8000/predict
El servidor Python devuelve una respuesta con los cálculos.

WebSockets con websockets en Python

Ventaja: Más rápido para comunicación en tiempo real.
Desventaja: Requiere un poco más de código en Godot y Python.
Ejemplo:
Godot establece una conexión WebSocket con el servidor Python en ws://127.0.0.1:8765.
El servidor recibe comandos y envía respuestas de inmediato.

Sockets TCP/UDP personalizados

Ventaja: Mayor flexibilidad y velocidad para grandes volúmenes de datos.
Desventaja: Más trabajo de implementación.

Ejemplo:
Se usa socket en Python y StreamPeerTCP en Godot.


📌 Arquitectura recomendada

Godot (Front-end y Simulación de Agentes)

Define los agentes, visualización y comportamiento básico.
Se comunica con el servidor Python para obtener decisiones de IA.
Servidor Python (IA y Cálculo)

Corre localmente en 127.0.0.1 y procesa datos.
Usa TensorFlow, PyTorch o Scikit-learn para predicciones.
Devuelve respuestas a Godot en tiempo real.
Usaremos **WebSockets**. **WebSockets** son ideales para **comunicación en tiempo real**, lo que es clave si queremos que los agentes en Godot interactúen dinámicamente con la IA en Python.  

---

### **📌 Pasos para Implementarlo**
1. **Configurar un servidor WebSocket en Python**  
   - Usaremos la librería `websockets` para manejar la conexión.  
   - El servidor procesará los datos y enviará respuestas en tiempo real.  

2. **Configurar WebSockets en Godot**  
   - Usaremos la clase `WebSocketClient` en GDScript.  
   - Godot se conectará al servidor y enviará/recibirá información.  

---

### **📌 Recursos Necesarios**
🔹 **Python**:  
   - `websockets` (`pip install websockets`)  
   - `asyncio` para manejar la comunicación de manera asíncrona  
   - Librerías como `numpy`, `torch`, etc., si se usa IA  

🔹 **Godot 4.4**:  
   - `WebSocketClient` en GDScript  
   - Scripts para manejar el envío/recepción de datos  

🔹 **Opcionales** (si se usa IA avanzada):  
   - TensorFlow o PyTorch  
   - FastAPI (si en el futuro queremos una REST API junto con WebSockets)  

---

### **📌 ¿Cómo Funcionará?**
1. **Godot inicia la conexión WebSocket** a `ws://127.0.0.1:8765`.  
2. **El servidor Python recibe los datos, los procesa** (ejemplo: simula IA de agentes).  
3. **Python envía la respuesta a Godot**, que actualiza los agentes en tiempo real.  

---



 💡 Siguiente paso: estructura básica del código para ambos lados. 🚀


Genial, vamos a estructurar ambos lados:  

1. **Servidor WebSocket en Python**  
2. **Cliente WebSocket en Godot (GDScript)**  

---

## **1️⃣ Servidor WebSocket en Python**
Este servidor manejará la comunicación con Godot y procesará los datos.  

📌 **Instala la librería necesaria** (si no la tienes):  
```sh
pip install websockets asyncio
```

📜 **Código en Python (`server.py`)**
```python
import asyncio
import websockets
import json

# Función para manejar mensajes de Godot
async def handle_message(websocket, path):
    async for message in websocket:
        print(f"Mensaje recibido: {message}")
        
        # Simulación de respuesta (aquí podrías agregar IA o lógica avanzada)
        response = {"status": "ok", "message": "Respuesta desde Python"}
        
        # Enviar respuesta a Godot
        await websocket.send(json.dumps(response))

# Iniciar servidor WebSocket en el puerto 8765
async def start_server():
    server = await websockets.serve(handle_message, "localhost", 8765)
    print("Servidor WebSocket iniciado en ws://localhost:8765")
    await server.wait_closed()

# Ejecutar el servidor
asyncio.run(start_server())
```

🔹 **Este código:**  
✅ Escucha conexiones en `ws://localhost:8765`  
✅ Recibe mensajes de Godot y responde con un JSON  

---

## **2️⃣ Cliente WebSocket en Godot (GDScript)**
📜 **Código en Godot (`websocket.gd`)**
```gdscript
extends Node

var websocket = WebSocketClient.new()

func _ready():
    # Conectar señales del WebSocket
    websocket.connect("connection_established", Callable(self, "_on_connected"))
    websocket.connect("data_received", Callable(self, "_on_message"))
    websocket.connect("connection_closed", Callable(self, "_on_disconnected"))

    # Intentar conectar al servidor WebSocket en Python
    var err = websocket.connect_to_url("ws://localhost:8765")
    if err != OK:
        print("Error al conectar WebSocket: ", err)
    else:
        print("Intentando conectar al servidor...")

func _process(delta):
    websocket.poll()  # Mantener conexión viva

func _on_connected(_protocol):
    print("Conectado al servidor WebSocket!")
    send_message("Hola desde Godot!")

func send_message(msg):
    var message = JSON.stringify({"message": msg})
    websocket.send_text(message)

func _on_message():
    var message = websocket.get_peer(1).get_packet().get_string_from_utf8()
    print("Mensaje recibido desde Python: ", message)

func _on_disconnected(_was_clean):
    print("Conexión cerrada")
```

🔹 **Este código en Godot:**  
✅ Se conecta al WebSocket en `ws://localhost:8765`  
✅ Envía un mensaje cuando se conecta  
✅ Recibe y muestra respuestas de Python  

---

## **🎯 Prueba el Sistema**
1. **Ejecuta el servidor en Python**  
   ```sh
   python server.py
   ```
   *Verás el mensaje "Servidor WebSocket iniciado en ws://localhost:8765"*

2. **Ejecuta Godot y corre la escena**  
   *Godot intentará conectarse y enviar un mensaje a Python*

3. **Revisa la consola** en ambos lados:  
   - Python debería mostrar `"Mensaje recibido: ..."`.  
   - Godot debería recibir la respuesta y mostrar `"Mensaje recibido desde Python: ..."`.  

---

💡 **¿Qué sigue?**  
Ahora podrás extender esto para enviar información más compleja, como datos de agentes de IA, comandos de movimiento, etc. 🚀


 Buena suerte!!
