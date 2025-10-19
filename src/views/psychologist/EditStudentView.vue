<!--
Componente EditStudentView

Este componente permite a los psicólogos editar la información de un estudiante.
Características:
- Formulario dividido en secciones (Información Personal y Académica)
- Validación de campos requeridos
- Manejo de estados de carga
- Notificaciones de éxito/error usando SweetAlert2
-->

<template>
<div class="student-details-container p-4 md:p-8 lg:p-10 min-h-screen bg-gray-50">
    <link href="https://cdn.jsdelivr.net/npm/remixicon@3.5.0/fonts/remixicon.css" rel="stylesheet">

    <div class="header-section mb-8 flex flex-col md:flex-row md:justify-between md:items-center border-b pb-4">
      <h1 class="text-3xl font-extrabold text-indigo-700 tracking-tight mb-4 md:mb-0">
        ✍️ Editar Estudiante
      </h1>
      <button 
        class="back-btn flex items-center px-4 py-2 text-base font-medium text-gray-700 bg-white border border-gray-300 rounded-lg shadow-sm hover:bg-gray-100 transition duration-150" 
        @click="goBack"
      >
        <i class="ri-arrow-left-line mr-2"></i> Volver
      </button>
    </div>

    <div v-if="loading" class="loading-state flex flex-col items-center justify-center py-20 text-indigo-600">
      <i class="ri-loader-4-line loading-spinner text-4xl animate-spin"></i>
      <p class="mt-3 text-lg font-medium">Cargando datos del estudiante...</p>
    </div>

    <div v-else class="edit-form bg-white rounded-xl shadow-xl p-6 lg:p-8">
      <form @submit.prevent="handleSubmit" class="student-form space-y-8">

        <div class="form-section">
          <h3 class="text-xl font-bold text-gray-800 border-b pb-3 mb-6 flex items-center">
            <i class="ri-user-line mr-3 text-indigo-500"></i> Información Personal
          </h3>
          <div class="form-grid grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            
            <div class="form-group">
              <label for="firstName" class="form-label">Nombre *</label>
              <input type="text" id="firstName" v-model="student.firstName" required class="form-input" />
            </div>

            <div class="form-group">
              <label for="lastName" class="form-label">Apellido *</label>
              <input type="text" id="lastName" v-model="student.lastName" required class="form-input" />
            </div>

            <div class="form-group">
              <label for="age" class="form-label">Edad *</label>
              <input type="number" id="age" v-model="student.age" required class="form-input" min="1" max="100" />
            </div>
            
            <div class="form-group">
              <label for="phone" class="form-label">Teléfono</label>
              <input type="tel" id="phone" v-model="student.phone" class="form-input" placeholder="+593 99 999 9999" />
            </div>

            <div class="form-group">
              <label for="city" class="form-label">Ciudad</label>
              <input type="text" id="city" v-model="student.city" class="form-input" placeholder="Ej: Guayaquil" />
            </div>
            
          </div>
        </div>

        ---

        <div class="form-section pt-4">
          <h3 class="text-xl font-bold text-gray-800 border-b pb-3 mb-6 flex items-center">
            <i class="ri-graduation-cap-line mr-3 text-indigo-500"></i> Información Académica
          </h3>
          <div class="form-grid grid grid-cols-1 md:grid-cols-2 gap-6">
            

            <div class="form-group">
              <label for="career" class="form-label">Carrera</label>
              <input
                type="text"
                id="career"
                v-model="student.career"
                class="form-input"
                placeholder="Ej: Psicología Clínica"
              />
            </div>

            </div>
        </div>

        ---

        <div class="form-actions pt-6 flex justify-end gap-4">
          <button type="button" class="btn-cancel" @click="goBack">
            <i class="ri-close-line"></i> Cancelar
          </button>
          <button type="submit" class="btn-save">
            <i class="ri-save-line"></i> Guardar Cambios
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Swal from 'sweetalert2';
const API_URL = import.meta.env.VITE_API_BASE_URL;
export default {
  /**
   * Nombre del componente para su identificación en Vue DevTools
   */
  name: 'EditStudentView',
  /**
   * Estado local del componente
   * @returns {Object} Estado inicial con datos del estudiante y estado de carga
   */
  data() {
    return {
      loading: true,
      student: {
        firstName: '',
        lastName: '',
        age: '',
        phone: '',
        city: '',
        level: '',
        career: ''
      }
    };
  },
  /**
   * Ciclo de vida: Se ejecuta cuando el componente es creado
   * Carga los detalles del estudiante desde el servidor
   */
  async created() {
    await this.fetchStudentDetails();
  },
  methods: {
    /**
     * Obtiene los detalles del estudiante desde el servidor
     * Utiliza el ID del estudiante desde los parámetros de la ruta
     * Actualiza el estado local con la información recibida
     */
    async fetchStudentDetails() {
      try {
        const token = localStorage.getItem('x-token');
        const response = await axios.get(
          `${API_URL}/api/students/${this.$route.params.id}`,
          {
            headers: {
              'x-token': token
            }
          }
        );
        this.student = response.data;
        this.loading = false;
      } catch (error) {
        console.error('Error al cargar los detalles del estudiante:', error);
        Swal.fire({
          icon: 'error',
          title: 'Error',
          text: 'No se pudo cargar la información del estudiante'
        });
        this.goBack();
      }
    },
    /**
     * Maneja el envío del formulario
     * Envía los datos actualizados al servidor
     * Muestra notificaciones de éxito/error
     * Redirige al detalle del estudiante si la actualización es exitosa
     */
    async handleSubmit() {
      try {
        const token = localStorage.getItem('x-token');
        const response = await axios.put(
          `${API_URL}/api/students/${this.$route.params.id}`,
          this.student,
          {
            headers: {
              'x-token': token
            }
          }
        );

        await Swal.fire({
          icon: 'success',
          title: 'Guardado',
          text: response.data.msg || 'Los cambios han sido guardados exitosamente'
        });

        this.$router.push(`/psychologist/dashboard/patients/${this.$route.params.id}`);
      } catch (error) {
        console.error('Error:', error);
        Swal.fire({
          icon: 'error',
          title: 'Error',
          text: 'No se pudieron guardar los cambios'
        });
      }
    },
    /**
     * Navega de vuelta a la vista de detalles del estudiante
     * Utiliza el ID del estudiante de los parámetros de la ruta
     */
    goBack() {
      this.$router.push(`/psychologist/dashboard/patients/${this.$route.params.id}`);
    }
  }
};
</script>

<style>
    /* Estilos de Fuente y Labels */
    .form-label {
      @apply block text-sm font-semibold text-gray-700 mb-1;
    }

    /* Estilos de Input (Unificados) */
    .form-input {
      @apply block w-full px-4 py-2 text-base text-gray-900 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 transition duration-150 ease-in-out placeholder-gray-400 shadow-sm;
    }

    /* Botón Principal (Guardar) */
    .btn-save {
      @apply flex items-center px-6 py-3 rounded-xl bg-indigo-600 text-white hover:bg-indigo-700 transition duration-150 ease-in-out font-semibold shadow-md hover:shadow-lg disabled:opacity-50 disabled:cursor-not-allowed;
    }

    /* Botón Secundario (Cancelar) */
    .btn-cancel {
      @apply flex items-center px-6 py-3 rounded-xl border border-gray-300 text-gray-700 bg-white hover:bg-gray-50 transition duration-150 ease-in-out font-medium;
    }

    /* Animación del Spinner */
    @keyframes spin {
      to { transform: rotate(360deg); }
    }
    .loading-spinner {
      animation: spin 1s linear infinite;
    }
  </style>