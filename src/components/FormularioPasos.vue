<template>
  <div>
    <div class="card shadow">
      <div class="card-header bg-primary text-white">
        <h2 class="h5">Formulario de satisfacción</h2>
      </div>

      <div class="card-body">
        <div class="progress" role="progressbar" aria-label="Basic example" aria-valuenow="0" aria-valuemin="0"
          aria-valuemax="100">
          <div class="progress-bar progress-bar-striped progress-bar-animated"
            :style="{ width: (pasoActual / totalPasos) * 100 + '%' }"></div>
        </div>

        <form @submit.prevent="enviarEncuesta">
          <template v-if="pasoActual === 1">
            <h3 class="h6 border-bottom pb-2">1. Datos personales</h3>
            <div class="mb-3">
              <label for="nombre" class="form-label">Nombre Completo</label>
              <input class="form-control" type="text" name="nombre" id="nombre" v-model="encuesta.nombre">
              <small v-if="errores.nombre" class="text-danger">{{ errores.nombre }}</small>
            </div>
            <div class="mb-3">
              <label for="email" class="form-label">Email</label>
              <input class="form-control" type="email" name="email" id="email" v-model="encuesta.email">
              <small v-if="errores.email" class="text-danger">{{ errores.email }}</small>
            </div>
            <div class="mb-3">
              <label for="edad" class="form-label">Edad</label>
              <input class="form-control" type="number" name="edad" id="edad" v-model.number="encuesta.edad">
              <small v-if="errores.edad" class="text-danger">{{ errores.edad }}</small>
            </div>
          </template>
          <template v-if="pasoActual === 2">
            <h3 class="h6 border-bottom pb-2">2. Intereses</h3>
            <p class="small text-muted">Selecciona al menos una opción:</p>
            <div class="form-check" v-for="opcion in opcionesIntereses" :key="opcion">
              <input type="checkbox" class="form-check-input" :id="opcion" :value="opcion" v-model="encuesta.intereses">
              <label :for="opcion">{{ opcion }}</label>
            </div>
            <small v-if="errores.intereses" class="text-danger">{{ errores.intereses }}</small>
          </template>
          <template v-if="pasoActual === 3">
            <h3 class="h6 border-bottom pb-2">3. Valoración</h3>
            <div class="mb-3">
              <label for="satisfaccion">Nivel de satisfacción (1 - 10): {{ encuesta.satisfaccion }}</label>
              <input type="range" class="form-range" id="satisfaccion" min="1" max="10" v-model="encuesta.satisfaccion">
            </div>
            <div class="mb-3">
              <label for="recomeinda">¿Recomendarías el bootcamp?</label>
              <div class="form-check">
                <input class="form-check-input" type="radio" name="si" id="si" value="si" v-model="encuesta.recomienda">
                <label class="form-check-label" for="si">
                  Si
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" type="radio" name="no" id="no" value="no" v-model="encuesta.recomienda">
                <label class="form-check-label" for="no">
                  No
                </label>
              </div>
            </div>
          </template>
          <template v-if="pasoActual === 4">
            <h3 class="h6 border-bottom pb-2">4. Comentarios</h3>
            <div class="mb-3">
              <label for="comentarios" class="form-label">Cuéntanos sobre tu experiencia</label>
              <textarea name="comentarios" id="comentarios" v-model="encuesta.comentarios" class="form-control"
                maxlength="250"></textarea>
              <p class="small text-muted text-end">{{ encuesta.comentarios.length }}/250</p>
              <small v-if="errores.comentarios" class="text-danger">{{ errores.comentarios }}</small>
            </div>
            <div class="mb-3">
              <label for="aceptaContacto">¿Aceptas que te llamemos para consultar algunos ítems de la encuesta?</label>
              <div class="form-check">
                <input class="form-check-input" type="radio" name="si" id="si" value="si"
                  v-model="encuesta.aceptaContacto">
                <label class="form-check-label" for="si">
                  Si
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" type="radio" name="no" id="no" value="no"
                  v-model="encuesta.aceptaContacto">
                <label class="form-check-label" for="no">
                  No
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" type="radio" name="jamas" id="jamas" value="jamas"
                  v-model="encuesta.aceptaContacto">
                <label class="form-check-label" for="jamas">
                  Jamás
                </label>
              </div>
            </div>
          </template>
          <div class="mb-3 d-flex justify-content-between" v-if="!enviado">
            <button type="button" class="btn btn-secondary" @click="pasoAnterior">Anterior</button>
            <button type="button" class="btn btn-primary" @click="pasoSiguiente"
              v-if="pasoActual < totalPasos">Siguiente</button>
            <button type="submit" class="btn btn-success" v-else>Enviar Encuesta</button>
          </div>

          <div v-else class="alert alert-success my-3">
            <h3>Encuesta enviada con éxito</h3>
            <hr>
            <ul class="list-unstyled">
              <li><strong>Usuario:</strong> {{ encuesta.nombre }} {{ encuesta.email }}</li>
              <li><strong>Edad:</strong> {{ encuesta.edad }}</li>
              <li><strong>Intereses:</strong> {{ encuesta.intereses.join(', ') }}</li>
              <li><strong>Satisfacción:</strong> {{ encuesta.satisfaccion }}</li>
              <li><strong>Recomienda:</strong> {{ encuesta.recomienda }}</li>
              <li><strong>Comentarios:</strong> {{ encuesta.comentarios }}</li>
            </ul>
            <button type="button" class="btn btn-primary" @click="download">Descargar datos del formulario</button>
          </div>
        </form>
      </div>

    </div>

  </div>
</template>

<script>
export default {
  name: 'FormularioPasos',
  // props: {},
  data() {
    return {
      encuesta: {
        nombre: '',
        email: '',
        edad: null,
        intereses: [],
        satisfaccion: 5,
        recomienda: 'si',
        comentarios: '',
        aceptaContacto: 'si'
      },
      pasoActual: 1,
      totalPasos: 4,
      enviado: false,
      errores: {
        nombre: '',
        email: '',
        edad: '',
        intereses: '',
        comentarios: ''
      },
      opcionesIntereses: ['Programación', 'IA', 'Bases de Datos', 'Frontend', 'Ninguna']
    }
  },
  // computed: {},
  methods: {
    validarPaso() {
      let valido = true // flag o bandera para determinar si formulario es valido o no
      this.limpiarErrores() // antes de validar, nos aseguramos de que no hayan errores antiguos almacenados

      if (this.pasoActual === 1) {
        if (!this.encuesta.nombre) {
          this.errores.nombre = 'El nombre es obligatorio'
          valido = false
        }

        if (!this.validarEmail(this.encuesta.email)) {
          this.errores.email = 'Ingresa un correo válido'
          valido = false
        }

        if (this.encuesta.edad < 18) {
          this.errores.edad = 'Debes ser mayor de edad'
          valido = false
        }
      }

      if (this.pasoActual === 2) {
        if (this.encuesta.intereses.length === 0) {
          this.errores.intereses = 'Selecciona al menos un interés'
          valido = false
        }
      }

      if (this.pasoActual === 4) {
        if (this.encuesta.comentarios.length < 10) {
          this.errores.comentarios = 'Cuéntanos un poco más (mínimo 10 caracteres)'
          valido = false
        }
      }

      return valido
    },

    validarEmail(email) {
      const regexEmail = /[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?/

      return regexEmail.test(email) // devuelve true si el email coincide con el patron esperado
    },
    limpiarErrores() {
      this.errores = {
        nombre: '',
        email: '',
        edad: '',
        intereses: '',
        comentarios: ''
      }
    },
    pasoAnterior() {
      if (this.pasoActual > 1) {
        this.pasoActual--
      }
    },
    pasoSiguiente() {
      if (this.pasoActual < 4 && this.validarPaso()) {
        this.pasoActual++
      }
    },
    enviarEncuesta() {
      if (this.validarPaso()) {
        console.log('Datos enviados:', this.encuesta)
        this.enviado = true
      }
    },
    // Método para descargar los datos del formulario como un archivo JSON
    download() {
      // JSON.stringify() convierte el objeto JavaScript en una cadena de texto en formato JSON
      // El segundo parámetro (null) indica que no aplicaremos filtros
      // El tercer parámetro (2) es para dar formato legible con 2 espacios de indentación
      const jsonContent = JSON.stringify(this.encuesta, null, 2)

      // Blob (Binary Large Object) es un objeto que representa datos en formato binario
      // Aquí envolvemos el contenido JSON con el tipo MIME 'application/json'
      // Esto le dice al navegador que es un archivo JSON
      const blob = new Blob([jsonContent], { type: 'application/json' })

      // Llamamos al método triggerDownload para manejar la descarga del archivo
      // Le pasamos el blob con los datos y el nombre que tendrá el archivo
      this.triggerDownload(blob, 'encuesta.json')
    },

    triggerDownload(blob, fileName) {
      // Paso 1: Crear una URL temporal que representa el blob (archivo) en memoria
      // URL.createObjectURL() genera una URL única que apunta a nuestro archivo
      // Esta URL solo existe mientras la página esté abierta
      const url = URL.createObjectURL(blob);

      // Paso 2: Crear un elemento <a> (anchor/enlace) de forma invisible
      // Aunque no lo veremos en la página, lo usaremos para simular una descarga
      const link = document.createElement('a');

      // Asignar la URL temporal al atributo href del enlace
      // De esta forma el enlace "apunta" a nuestro archivo
      link.href = url;

      // El atributo download es lo que hace que el navegador descargue el archivo
      // en lugar de abrirlo. El valor es el nombre con el que se guardará
      link.download = fileName;

      // Paso 3: Agregar el elemento al documento HTML para poder interactuar con él
      // Aunque no aparezca visualmente, necesita estar en el DOM para que funcione
      document.body.appendChild(link);

      // Gatillar un click en el enlace, esto dispara la descarga del archivo
      // Es como si el usuario hubiera hecho click manualmente
      link.click();

      // Paso 4: Limpiar los recursos que creamos para no contaminar el DOM
      // Eliminar el elemento <a> del documento porque ya no lo necesitamos
      document.body.removeChild(link);

      // Liberar la memoria de la URL temporal
      // Esto es muy importante para evitar pérdidas de memoria (memory leaks)
      // El navegador mantiene un registro de estas URLs hasta que las revocamos
      URL.revokeObjectURL(url);
    }
  },
  // watch: {},
  // components: {},
  // mixins: [],
  // filters: {},
  // -- Lifecycle Methods
  // -- End Lifecycle Methods
}
</script>

<style scoped></style>