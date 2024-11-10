<template>
  <div id="container">
    <div id="tittleContainer">
      <h3>👋Detección de Lengua de Señas Paraguay</h3>
    </div>

    <div id="switchContainer">
      <label for="cameraSwitch" class="camera-label">📸</label>
      <label class="switch">
        <input type="checkbox" v-model="isCameraOn" @change="toggleCamera">
        <span class="slider"></span>
      </label>
    </div>

    <div id="videoContainer">
      <video id="video" ref="video" autoplay></video>
      <canvas ref="canvas" style="display: none;"></canvas>
    </div>

    <div id="resultsContainer" v-if="isCameraOn">
      <h4>🔍 Traducción detectada:</h4>
      <p id="sentence">{{ currentSentence }}</p>
    </div>

    <p><strong>📜Instrucciones:</strong><br>
      🔹 Activa el interruptor <strong>📸</strong> para comenzar la detección de señas.<br>
      🔹 Asegúrate de permitir el acceso a la cámara cuando el navegador lo solicite.</p>
  </div>
</template>

<script>
import { io } from 'socket.io-client';

export default {
  name: 'ComponenteCamara',

  data() {
    return {
      isCameraOn: false,
      stream: null,
      socket: null,
      frameInterval: null,
      currentSentence: '',
      isProcessing: false
    };
  },

  created() {
    // Inicializar Socket.IO
    this.socket = io('http://localhost:5000');

    // Configurar los listeners del socket
    this.setupSocketListeners();
  },

  beforeUnmount() {
    this.cleanup();
  },

  methods: {
    setupSocketListeners() {
      this.socket.on('connect', () => {
        console.log('Conectado al servidor');
      });

      this.socket.on('prediction', (data) => {
        this.currentSentence = data.sentence.join(' | ');
        console.log('Resultado:', this.currentSentence);
      });

      this.socket.on('error', (data) => {
        console.error('Error del servidor:', data.message);
      });
    },

    async toggleCamera() {
      if (this.isCameraOn) {
        await this.startCamera();
      } else {
        this.stopCamera();
      }
    },

    async startCamera() {
      const videoElement = this.$refs.video;

      try {
        // Solicitar acceso a la cámara
        this.stream = await navigator.mediaDevices.getUserMedia({
          video: {
            width: { ideal: 640 },
            height: { ideal: 480 },
            frameRate: { ideal: 30 }
          }
        });

        // Configurar el video
        videoElement.srcObject = this.stream;
        await videoElement.play();

        // Iniciar el procesamiento de frames
        this.startFrameProcessing();
      } catch (err) {
        console.error("Error al acceder a la cámara: ", err);
        this.isCameraOn = false;
      }
    },

    stopCamera() {
      // Detener el procesamiento de frames
      if (this.frameInterval) {
        clearInterval(this.frameInterval);
        this.frameInterval = null;
      }

      // Detener todos los tracks de la cámara
      if (this.stream) {
        this.stream.getTracks().forEach(track => track.stop());
        this.$refs.video.srcObject = null;
        this.stream = null;
      }

      // Limpiar el resultado actual
      this.currentSentence = '';
    },

    startFrameProcessing() {
      const canvas = this.$refs.canvas;
      const video = this.$refs.video;
      const context = canvas.getContext('2d');

      // Configurar el tamaño del canvas
      canvas.width = 640;
      canvas.height = 480;

      // Procesar frames cada 100ms (10 FPS)
      this.frameInterval = setInterval(() => {
        if (!this.isProcessing && this.isCameraOn) {
          this.isProcessing = true;

          // Dibujar el frame actual en el canvas
          context.drawImage(video, 0, 0, canvas.width, canvas.height);

          // Convertir el frame a base64 y enviarlo al servidor
          const frameData = canvas.toDataURL('image/jpeg', 0.8);
          this.socket.emit('frame', frameData);

          this.isProcessing = false;
        }
      }, 100);
    },

    cleanup() {
      // Detener la cámara
      this.stopCamera();

      // Desconectar el socket
      if (this.socket) {
        this.socket.disconnect();
        this.socket = null;
      }
    }
  }
};
</script>

<style scoped>
#container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

#tittleContainer {
  text-align: center;
  margin-bottom: 20px;
}

#switchContainer {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

#videoContainer {
  width: 100%;
  max-width: 640px;
  margin: 0 auto;
  overflow: hidden;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

#video {
  width: 100%;
  height: auto;
  transform: scaleX(-1);
  /* Espejo horizontal */
}

#resultsContainer {
  margin-top: 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 8px;
  text-align: center;
  color: #333;
}

#sentence {
  color: #000;
  font-size: 1.2em;
}

.camera-label {
  margin-right: 10px;
  font-size: 1.5em;
}

/* Estilos para el switch */
.switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: .4s;
  border-radius: 34px;
}

.slider:before {
  position: absolute;
  content: "";
  height: 26px;
  width: 26px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked+.slider {
  background-color: #2196F3;
}

input:checked+.slider:before {
  transform: translateX(26px);
}
</style>
