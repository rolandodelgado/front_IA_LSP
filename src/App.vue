<template>
  <div id="app">
    <!-- Llama al componente ComponenteCamara aquí -->
    <ComponenteCamara />
  </div>
</template>

<script>
import ComponenteCamara from './components/ComponenteCamara.vue'

export default {
  name: 'App',
  components: {
    ComponenteCamara
  }
}
</script>

<style>
/* Puedes agregar aquí estilos adicionales si los necesitas */
#app {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
</style>
