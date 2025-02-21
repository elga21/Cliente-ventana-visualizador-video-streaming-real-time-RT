# Cliente-ventana-visualizador-video-streaming-real-time-RT
Aplicacion para visualizar una transmision de video en una red mediante la conexion a un servidor en un puerto determinado

        "Esta es una aplicación que es un visualizador de video diseñado para recibir un video Streaming en tiempo real (RT), "
        "created by the developer: Fernando Bevans mail:fernando.bevans@gmail.com github elga21 " 
        "Esta aplicación  se conecta a un socket Udp al servidor por medio del puerto determinado, la aplicacion de despliegue como el ejecutable esta creado para linux "

        confirma el IP y el purto de conexión del seevidor, la aplicacion recibirá el video en el formato y tamaño que lo envie el servidor de straming

        
import cv2
import asyncio
import websockets
import numpy as np
import threading
import tkinter as tk
from tkinter import messagebox

# Función asíncrona para la conexión al servidor WebSocket
async def cliente(uri):
    try:
        async with websockets.connect(uri) as websocket:
            while True:
                try:
                    # Recibir el frame
                    frame_data = await websocket.recv()
                    frame = np.frombuffer(frame_data, dtype=np.uint8)
                    img = cv2.imdecode(frame, cv2.IMREAD_COLOR)
                    # Mostrar el frame
                    cv2.imshow("Video en Tiempo Real", img)
                    if cv2.waitKey(1) == ord('q'):
                        break
                except websockets.ConnectionClosed:
                    print("Conexión cerrada por el servidor.")
                    break
    except Exception as e:
        print(f"Ocurrió un error: {e}")
    finally:
        cv2.destroyAllWindows()

# Función que ejecuta la conexión dentro de un bucle asíncrono
def iniciar_cliente_conexion(ip, puerto):
    uri = f"ws://{ip}:{puerto}/ws"
    print(f"Conectando a {uri}...")
    try:
        asyncio.run(cliente(uri))
    except Exception as e:
        print("Error al iniciar el cliente:", e)

# Función que se ejecuta al presionar el botón "Conexión" en la interfaz
def conectar():
    ip = entry_ip.get().strip()
    puerto = entry_puerto.get().strip()
    if not ip or not puerto.isdigit():
        messagebox.showerror("Error", "Ingresa una IP y un puerto válidos.")
        return
    label_estado.config(text=f"Conectando a {ip}:{puerto} ...")
    hilo = threading.Thread(target=iniciar_cliente_conexion, args=(ip, puerto), daemon=True)
    hilo.start()
    label_estado.config(text="Conexión iniciada.")

# Función para mostrar la ayuda
def mostrar_ayuda():
    mensaje = (
        "Esta es una aplicación que es un visualizador de video diseñado para recibir un video Streaming en tiempo real (RT), "
        "created by the developer: Fernando Bevans mail:fernando.bevans@gmail.com github elga21 " 
        "Esta aplicación  se conecta a un socket Udp al servidor por medio del puerto determinado, la aplicacion de despliegue como el ejecutable esta creado para linux "

    )
    messagebox.showinfo("Ayuda", mensaje)

# Configuración de la ventana principal de Tkinter
ventana = tk.Tk()
ventana.title("Cliente de Video Streaming")

# Campo para la IP
tk.Label(ventana, text="IP:").grid(row=0, column=0, padx=5, pady=5)
entry_ip = tk.Entry(ventana)
entry_ip.grid(row=0, column=1, padx=5, pady=5)
entry_ip.insert(0, "0.0.0.0")  # Valor por defecto

# Campo para el puerto
tk.Label(ventana, text="Puerto:").grid(row=1, column=0, padx=5, pady=5)
entry_puerto = tk.Entry(ventana)
entry_puerto.grid(row=1, column=1, padx=5, pady=5)
entry_puerto.insert(0, "7100")  # Valor por defecto

# Botón de Conexión
btn_conectar = tk.Button(ventana, text="Conexión", command=conectar)
btn_conectar.grid(row=2, column=0, columnspan=2, padx=5, pady=10)

# Botón de Ayuda
btn_ayuda = tk.Button(ventana, text="Ayuda", command=mostrar_ayuda)
btn_ayuda.grid(row=3, column=0, columnspan=2, padx=5, pady=5)

# Etiqueta para mostrar el estado
label_estado = tk.Label(ventana, text="")
label_estado.grid(row=4, column=0, columnspan=2, padx=5, pady=5)

ventana.mainloop()
