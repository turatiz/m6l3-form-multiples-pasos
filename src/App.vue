<script>
import FormularioPasos from './components/FormularioPasos.vue';
export default {
  name: 'App',
  data() {
    return {}
  },
  components: {
    FormularioPasos
  }
}
</script>

<template>
  <main class="container my-5">
    <div class="row justify-content-center">
      <div class="col-md-6">
        <FormularioPasos></FormularioPasos>
      </div>
    </div>
  </main>
</template>

<style scoped></style>
