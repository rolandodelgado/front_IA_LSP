<template>
  <div id="container">
    <div id="tittleContainer">
      <h3>👋Detección de Lengua de Señas Paraguay</h3>
    </div>

    <div id="switchContainer">
      <label for="cameraSwitch" class="camera-label">📸</label>
      <!-- Switch para activar/desactivar la cámara -->
      <label class="switch">
        <input type="checkbox" v-model="isCameraOn" @change="toggleCamera">
        <span class="slider"></span>
      </label>
    </div>

    <div id="videoContainer">
      <video id="video" ref="video" autoplay></video>
    </div>

    <p><strong>📜Instrucciones:</strong><br>
      🔹 Activa el interruptor <strong>📸</strong> para comenzar la detección de señas.<br>
      🔹 Asegúrate de permitir el acceso a la cámara cuando el navegador lo solicite.</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isCameraOn: false,
      stream: null,
    };
  },
  methods: {
    async toggleCamera() {
      const videoElement = this.$refs.video;

      if (this.isCameraOn) {
        try {
          this.stream = await navigator.mediaDevices.getUserMedia({ video: true });
          videoElement.srcObject = this.stream;
        } catch (err) {
          console.error("Error al acceder a la cámara: ", err);
        }
      } else {
        if (this.stream) {
          this.stream.getTracks().forEach(track => track.stop());
          videoElement.srcObject = null;
          this.stream = null;
        }
      }
    }
  }
};
</script>
