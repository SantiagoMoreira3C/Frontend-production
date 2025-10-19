<template>
  <div class="px-4 py-6 sm:px-8">
    <!-- Header -->
    <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-6">
      <div>
        <h2 class="text-2xl font-bold text-gray-800">Mis Pacientes</h2>
        <span class="text-sm text-gray-500">{{ users.length }} pacientes en total</span>
      </div>
      <div class="flex flex-col sm:flex-row sm:items-center gap-2">
        <div class="relative">
          <input
            type="text"
            v-model="searchTerm"
            @input="filterPatients"
            placeholder="Buscar paciente..."
            class="pl-10 pr-4 py-2 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 focus:outline-none w-full sm:w-64"
          />
          <i class="fas fa-search absolute left-3 top-2.5 text-gray-400"></i>
        </div>
        <button
          class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg flex items-center gap-2 shadow"
          @click="showModal = true"
        >
          <i class="fas fa-plus"></i> Agregar Estudiante
        </button>
      </div>
    </div>

    <!-- Modal -->
<div 
  v-if="showModal" 
  class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-gray-900 bg-opacity-75 backdrop-blur-sm transition-opacity duration-300
         /* CLASES MODIFICADAS: MAYOR DESPLAZAMIENTO A LA DERECHA */
         md:translate-x-24 lg:translate-x-40" 
>
  
  <div 
    class="bg-white rounded-2xl shadow-2xl max-w-4xl w-full max-h-[95vh] overflow-hidden flex flex-col transform transition-all duration-300 scale-100 opacity-100"
  >

    <div class="p-6 border-b border-gray-100 flex justify-between items-center sticky top-0 bg-white z-10">
      <h3 class="text-2xl font-extrabold text-black-700 font-sans tracking-tight">
        {{ selectedStudent ? '✍️ Editar' : '✨ Agregar' }} Estudiante
      </h3>
      <button @click="closeModal" class="text-gray-400 hover:text-indigo-600 transition-colors duration-200 p-1 rounded-full hover:bg-indigo-50" aria-label="Cerrar modal">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
      </button>
    </div>

    <div class="p-6 overflow-y-auto custom-scrollbar">
      <form @submit.prevent="saveStudent" class="space-y-6">
        
        <h4 class="text-lg font-semibold text-gray-800 border-b pb-2 mb-4">Información Personal</h4>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Nombre <span class="text-red-500">*</span></label>
            <input v-model="formData.firstName" @input="validateField('firstName')" required placeholder="Ingrese el nombre" :class="{ 'border-red-500': formErrors.firstName }" class="form-input" />
            <p v-if="formErrors.firstName" class="text-red-500 text-xs mt-1">{{ formErrors.firstName }}</p>
          </div>
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Apellido <span class="text-red-500">*</span></label>
            <input v-model="formData.lastName" @input="validateField('lastName')" required placeholder="Ingrese el apellido" :class="{ 'border-red-500': formErrors.lastName }" class="form-input" />
            <p v-if="formErrors.lastName" class="text-red-500 text-xs mt-1">{{ formErrors.lastName }}</p>
          </div>
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Edad <span class="text-red-500">*</span></label>
            <input type="number" v-model.number="formData.age" @input="validateField('age')" min="0" max="100" required :class="{ 'border-red-500': formErrors.age }" class="form-input" />
            <p v-if="formErrors.age" class="text-red-500 text-xs mt-1">{{ formErrors.age }}</p>
          </div>
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Género <span class="text-red-500">*</span></label>
            <select v-model="formData.gender" @change="validateField('gender')" required :class="{ 'border-red-500': formErrors.gender }" class="form-select">
              <option value="" disabled>Seleccione género</option>
              <option value="MASCULINO">Masculino</option>
              <option value="FEMENINO">Femenino</option>
              <option value="OTRO">Otro</option>
            </select>
            <p v-if="formErrors.gender" class="text-red-500 text-xs mt-1">{{ formErrors.gender }}</p>
          </div>
        </div>

        <h4 class="text-lg font-semibold text-gray-800 border-b pb-2 mb-4 pt-4">Contacto y Ubicación</h4>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Email <span class="text-red-500">*</span></label>
            <input type="email" v-model="formData.email" @input="validateField('email')" required placeholder="ejemplo@dominio.com" :class="{ 'border-red-500': formErrors.email }" class="form-input" />
            <p v-if="formErrors.email" class="text-red-500 text-xs mt-1">{{ formErrors.email }}</p>
          </div>
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Teléfono <span class="text-red-500">*</span></label>
            <input v-model="formData.phone" @input="validateField('phone')" required placeholder="Ingrese el teléfono" :class="{ 'border-red-500': formErrors.phone }" class="form-input" />
            <p v-if="formErrors.phone" class="text-red-500 text-xs mt-1">{{ formErrors.phone }}</p>
          </div>
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Ciudad <span class="text-red-500">*</span></label>
            <input v-model="formData.city" @input="validateField('city')" required placeholder="Ingrese la ciudad" :class="{ 'border-red-500': formErrors.city }" class="form-input" />
            <p v-if="formErrors.city" class="text-red-500 text-xs mt-1">{{ formErrors.city }}</p>
          </div>
        </div>

        <h4 class="text-lg font-semibold text-gray-800 border-b pb-2 mb-4 pt-4">Académico y Laboral</h4>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Carrera <span class="text-red-500">*</span></label>
            <input v-model="formData.career" @input="validateField('career')" required placeholder="Ingrese carrera" :class="{ 'border-red-500': formErrors.career }" class="form-input" />
            <p v-if="formErrors.career" class="text-red-500 text-xs mt-1">{{ formErrors.career }}</p>
          </div>
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Semestre <span class="text-red-500">*</span></label>
            <select v-model="formData.careerLevel" @change="validateField('careerLevel')" required :class="{ 'border-red-500': formErrors.careerLevel }" class="form-select">
              <option value="" disabled>Seleccione semestre</option>
              <option value="PRIMERO">Primero</option>
              <option value="SEGUNDO">Segundo</option>
              <option value="TERCERO">Tercero</option>
              <option value="CUARTO">Cuarto</option>
              <option value="QUINTO">Quinto</option>
              <option value="SEXTO">Sexto</option>
              <option value="SEPTIMO">Séptimo</option>
              <option value="OCTAVO">Octavo</option>
              <option value="NOVENO">Noveno</option>
              <option value="DECIMO">Décimo</option>
            </select>
            <p v-if="formErrors.careerLevel" class="text-red-500 text-xs mt-1">{{ formErrors.careerLevel }}</p>
          </div>
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Estado Laboral <span class="text-red-500">*</span></label>
            <select v-model="formData.employmentStatus" @change="validateField('employmentStatus')" required :class="{ 'border-red-500': formErrors.employmentStatus }" class="form-select">
              <option value="" disabled>Seleccione estado</option>
              <option value="EMPLEADO">Empleado</option>
              <option value="DESEMPLEADO">Desempleado</option>
            </select>
            <p v-if="formErrors.employmentStatus" class="text-red-500 text-xs mt-1">{{ formErrors.employmentStatus }}</p>
          </div>
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Ingresos (USD)</label>
            <input type="number" v-model.number="formData.income" @input="validateField('income')" placeholder="Ingrese ingresos" :class="{ 'border-red-500': formErrors.income }" class="form-input" min="0" />
            <p v-if="formErrors.income" class="text-red-500 text-xs mt-1">{{ formErrors.income }}</p>
          </div>
        </div>
      </form>
    </div>

    <div class="p-6 border-t border-gray-100 flex justify-end gap-3 sticky bottom-0 bg-white z-10">
      <button type="button" @click="closeModal" class="px-5 py-2.5 rounded-xl border border-gray-300 text-gray-700 hover:bg-gray-50 transition duration-150 ease-in-out font-medium">
        Cancelar
      </button>
      <button type="submit" @click="saveStudent" class="px-5 py-2.5 rounded-xl [background-image:linear-gradient(180deg,#7f53ac_0%,#647dee_100%)] text-white hover:bg-indigo-700 transition duration-150 ease-in-out font-semibold shadow-md hover:shadow-lg disabled:opacity-50 disabled:cursor-not-allowed" :disabled="!isFormValid">
        <span v-if="selectedStudent">💾 Actualizar Estudiante</span>
        <span v-else  >➕ Crear Estudiante</span>
      </button>
    </div>
  </div>
</div>
    <!-- Tabla -->
<div v-else class="overflow-x-auto h-[calc(100vh-300px)]">
  <table class="w-full border-collapse">
    <thead>
      <tr class="bg-gray-100 text-gray-700 text-left sticky top-0 z-10">
        <th class="p-3">Estudiante</th>
        <th class="p-3">Email</th>
        <th class="p-3">Carrera</th>
        <th class="p-3 text-center">Acciones</th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="user in filteredUsers"
        :key="user._id"
        class="bg-white cursor-pointer border-b"
        @click="viewStudentDetails(user)"
      >
        <td class="p-3 ">
          <div class="flex items-center">
            <div class="min-w-[40px] min-h-[40px] w-10 h-10 flex items-center justify-center rounded-full bg-gradient-to-r from-blue-500 to-purple-600 text-white font-bold text-sm leading-none">
              {{ getInitials(`${user?.firstName || ''} ${user?.lastName || ''}`) }}
            </div>
            <div class="ml-3">
              <div class="text-sm font-medium text-gray-900">{{ user.firstName }} {{ user.lastName }}</div>
            </div>
          </div>
        </td>

        <td class="p-3">{{ user.email }}</td>
        <td class="p-3">{{ user.career }}</td>

        <td class="p-3 text-center">
          <div class="flex justify-center gap-2">
            <button class="text-blue-500 hover:text-blue-700" title="Editar" @click.stop="editStudent(user)">
              <i class="fas fa-edit"></i>
            </button>
            <button class="text-yellow-500 hover:text-yellow-600" title="Notas" @click.stop>
              <i class="fas fa-notes-medical"></i>
            </button>
            <button class="text-green-500 hover:text-green-600" title="Agendar" @click.stop>
              <i class="fas fa-calendar-plus"></i>
            </button>
          </div>
        </td>
      </tr>
    </tbody>
  </table>
</div>

    <!-- Importación -->
    <div class="mt-8">
      <h2 class="text-lg font-semibold mb-2">Importar Estudiantes</h2>
      <input type="file" @change="handleFileUpload" accept=".xlsx, .xls" class="mb-2 block" />
      <button
        class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
        @click="uploadExcel"
        :disabled="!selectedFile || isLoading"
      >
        {{ isLoading ? 'Subiendo...' : 'Importar' }}
      </button>
      <p v-if="uploadMessage" class="mt-2 text-green-600">{{ uploadMessage }}</p>
      <p v-if="uploadError" class="mt-2 text-red-600">{{ uploadError }}</p>
    </div>
    <div class="mt-8">
      <h2 class="text-lg font-semibold mb-2">Descargar Plantilla para ingresar usuarios en masa (Lea los comentarios de cada columna en el excel.)</h2>
      <button
        class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
        @click="downloadTemplate"
      >
      Descargar Plantilla
      <a 
     ref="downloadLink" 
     href="/plantilla_usuarios.xlsx" 
     download="plantilla_usuarios.xlsx" 
     style="display: none;"
   ></a>
      </button>
    </div>
  </div>
</template>

<style scoped>
.input {
  @apply w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500;
}
</style>
<script>
const API_URL = import.meta.env.VITE_API_BASE_URL;
/**
 * Importación de dependencias
 * @requires axios - Cliente HTTP para realizar peticiones al servidor
 * @requires sweetalert2 - Biblioteca para mostrar notificaciones elegantes
 */
import axios from 'axios'
import Swal from 'sweetalert2'

/**
 * Componente para gestionar la lista de pacientes del psicólogo
 * @component
 */
export default {
  name: 'PatientsView',
  /**
   * Estado local del componente
   * @returns {Object} Estado inicial con datos de pacientes, formulario y validación
   */
  data() {
    return {
      selectedFile: null,
      uploadMessage: '',
      uploadError: '',
      isLoading: false,
      users: [],
      filteredUsers: [],
      loading: true,
      error: null,
      searchTerm: '',
      showModal: false,
      selectedStudent: null,
      formData: {
        firstName: '',
        lastName: '',
        age: '',
        city: '',
        phone: '',
        gender: '',
        careerLevel: '',
        employmentStatus: '',
        income: '',
        career: '',
        email: '',
        finalRemarks: ''
      },
      formErrors: {}
    }
  },
  /**
   * Ciclo de vida: Se ejecuta cuando el componente es creado
   * Carga la lista inicial de pacientes
   */
  
  async created() {
    await this.fetchUsers()
  },
  /**
   * Métodos del componente para manejar la lógica de negocio
   */
  methods: {
    downloadTemplate() {
      // Simplemente simulamos un clic en el enlace oculto
      this.$refs.downloadLink.click();
    },
    getInitials(name) {
  try {
    if (!name || typeof name !== 'string') return '';

    const parts = name.trim().split(/\s+/).filter(Boolean);

    const first = parts?.[0]?.[0]?.toUpperCase() || '';
    const second = parts?.[1]?.[0]?.toUpperCase() || '';

    return first + second;
  } catch (error) {
    console.error('Error en getInitials:', error);
    return '';
  }
},
    handleFileUpload(event) {
      this.selectedFile = event.target.files[0];
      this.uploadMessage = '';
      this.uploadError = '';
    },
    async uploadExcel() {
      if (!this.selectedFile) return;

      this.isLoading = true;
      this.uploadMessage = '';
      this.uploadError = '';

      const formData = new FormData();
      formData.append('file', this.selectedFile);

      try {
        const token = localStorage.getItem('x-token');

        const response = await axios.post(`${API_URL}/api/students/import`, formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
            'x-token': token
          }
        });

        this.uploadMessage = response.data.msg || 'Estudiantes importados con éxito';
        this.selectedFile = null;
      } catch (error) {
        console.error(error);
        this.uploadError = error.response?.data?.msg || 'Error al importar estudiantes';
      } finally {
        this.isLoading = false;
      }
    },
    
    /**
     * Obtiene la lista de pacientes desde el servidor
     * @async
     */
    async fetchUsers() {
      try {
        this.loading = true
        this.error = null
        const token = localStorage.getItem('x-token')
        const response = await axios.get(`${API_URL}/api/students`, {
          headers: {
            'x-token': token
          }
        })

        this.users = response.data.students
        this.filteredUsers = this.users
      } catch (err) {
        this.error = 'Error al cargar los pacientes: ' + (err.response?.data?.message || err.message)
        console.error('Error:', err)
      } finally {
        this.loading = false
      }
    },
    /**
     * Genera las iniciales del nombre completo del paciente
     * @param {string} firstName - Nombre del paciente
     * @param {string} lastName - Apellido del paciente
     * @returns {string} Iniciales en mayúsculas
     */

    /**
     * Filtra la lista de pacientes según el término de búsqueda
     * Actualiza la lista filtrada en tiempo real
     */
    filterPatients() {
      const searchLower = this.searchTerm.toLowerCase()
      this.filteredUsers = this.users.filter(user => 
        user.firstName.toLowerCase().includes(searchLower) ||
        user.lastName.toLowerCase().includes(searchLower)
      )
    },
    /**
     * Cierra el modal y reinicia el formulario
     * Limpia los errores de validación
     */
    closeModal() {
      this.showModal = false
      this.selectedStudent = null
      this.formData = {
        firstName: '',
        lastName: '',
        age: '',
        city: '',
        phone: '',
        gender: '',
        careerLevel: '',
        employmentStatus: '',
        income: '',
        career: '',
        email: '',
      }
      this.formErrors = {}
    },
    /**
     * Valida un campo específico del formulario
     * @param {string} field - Nombre del campo a validar
     */
    validateField(field) {
      this.formErrors[field] = ''

      const value = this.formData[field]?.toString().trim() || ''
      const age = field === 'age' ? parseInt(value) : null
      const income = field === 'income' && value !== '' ? parseFloat(value) : null

      switch (field) {
        case 'firstName':
        case 'lastName':
          if (!value) {
            this.formErrors[field] = `El ${field === 'firstName' ? 'nombre' : 'apellido'} es requerido`
          } else if (!/^[a-zA-ZáéíóúÁÉÍÓÚñÑ\s]{2,30}$/.test(value)) {
            this.formErrors[field] = `El ${field === 'firstName' ? 'nombre' : 'apellido'} solo debe contener letras y espacios (2-30 caracteres)`
          }
          break

        case 'age':
          if (!value) {
            this.formErrors.age = 'La edad es requerida'
          } else if (isNaN(age) || age < 0 || age > 100) {
            this.formErrors.age = 'La edad debe estar entre 0 y 100 años'
          }
          break

        case 'city':
          if (!value) {
            this.formErrors.city = 'La ciudad es requerida'
          } else if (value.length < 2 || value.length > 50) {
            this.formErrors.city = 'La ciudad debe tener entre 2 y 50 caracteres'
          }
          break

        case 'phone':
          if (!value) {
            this.formErrors.phone = 'El teléfono es requerido'
          } else if (!/^\+?[\d\s-]{6,15}$/.test(value)) {
            this.formErrors.phone = 'Formato de teléfono inválido'
          }
          break

        case 'gender':
          if (!value) {
            this.formErrors.gender = 'El género es requerido'
          }
          break

        case 'employmentStatus':
          if (!value) {
            this.formErrors.employmentStatus = 'El estado laboral es requerido'
          }
          break
        case 'careerLevel':
          if (!value) {
            this.formErrors.careerLevel = 'El Semestre es requerido'
          }
          break

        case 'income':
          if (value !== '') {
            if (isNaN(income) || income < 0) {
              this.formErrors.income = 'Los ingresos no pueden ser negativos'
            }
          }
          break

        case 'career':
          if (!value) {
            this.formErrors.career = 'La carrera es requerida'
          } else if (value.length < 2 || value.length > 50) {
            this.formErrors.career = 'La carrera debe tener entre 2 y 50 caracteres'
          }
          break

        case 'email':
          if (!value) {
            this.formErrors.email = 'El email es requerido'
          } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
            this.formErrors.email = 'Email inválido'
          }
          break
      }
    },
    /**
     * Guarda o actualiza la información de un paciente
     * @async
     * Realiza validación completa antes de enviar al servidor
     */
    async saveStudent() {
      const fieldsToValidate = Object.keys(this.formData).filter(field => 
        field !== 'password' || !this.selectedStudent
      )

      fieldsToValidate.forEach(field => {
        if (field !== 'income' || this.formData[field]) { // income es opcional
          this.validateField(field)
        }
      })

      const hasErrors = Object.values(this.formErrors).some(error => error !== '')

      if (hasErrors) {
        const errorMessages = Object.values(this.formErrors)
          .filter(error => error !== '')
          .join('\n')

        Swal.fire({
          icon: 'error',
          title: 'Error de validación',
          text: errorMessages
        })
        return
      }

      try {
        const token = localStorage.getItem('x-token')
        const url = `${API_URL}/api/students`
        const method = this.selectedStudent ? 'PUT' : 'POST'
        const endpoint = this.selectedStudent ? `${url}/${this.selectedStudent._id}` : url

        const dataToSend = { ...this.formData }
        if (this.selectedStudent) {
          delete dataToSend.password
        }

        const response = await axios({
          method,
          url: endpoint,
          headers: {
            'Content-Type': 'application/json',
            'x-token': token
          },
          data: dataToSend
        })

        const data = response.data

        if (response.status === 201 || response.status === 200) {
          Swal.fire({
            icon: 'success',
            title: 'Éxito',
            text: data.msg || (this.selectedStudent ? 'Estudiante actualizado correctamente' : 'Estudiante creado correctamente')
          })
          this.closeModal()
          await this.fetchUsers()
        } else {
          throw new Error(data.msg || 'Error al procesar la solicitud')
        }
      } catch (error) {
        console.error('Error:', error)
        Swal.fire({
          icon: 'error',
          title: 'Error',
          text: error.response?.data?.msg || error.message || 'Error al procesar la solicitud'
        })
      }
    },
    /**
     * Prepara el formulario para editar un paciente existente
     * @param {Object} student - Datos del paciente a editar
     */
    editStudent(student) {
      this.selectedStudent = student
      this.formData = {
        firstName: student.firstName,
        lastName: student.lastName,
        age: student.age,
        city: student.city,
        phone: student.phone,
        gender: student.gender,
        careerLevel: student.careerLevel,
        employmentStatus: student.employmentStatus,
        income: student.income,
        career: student.career,
        email: student.email,
        password: '' // No incluimos la contraseña en la edición
      }
      this.showModal = true
    },
    /**
     * Navega a la vista de detalles del paciente
     * @param {Object} student - Datos del paciente a visualizar
     */
    viewStudentDetails(student) {
      this.$router.push(`/psychologist/dashboard/patients/${student._id}`)
    }
  },
  /**
   * Propiedades computadas del componente
   */
  computed: {
    /**
     * Verifica si el formulario es válido para enviar
     * @returns {boolean} true si no hay errores de validación
     */
    isFormValid() {
      return Object.values(this.formErrors).every(error => error === '')
    }
  }
}
</script>

<style scoped> 
/* Estilos base para Inputs y Selects */
.form-input, .form-select {
  @apply block w-full px-4 py-2 text-base text-gray-900 border border-gray-300 rounded-xl focus:ring-indigo-500 focus:border-indigo-500 transition duration-150 ease-in-out placeholder-gray-400;
}

/* Estilo para Selects (Ajuste para la flecha) */
.form-select {
  /* Elimina el estilo de cursor predeterminado si es necesario */
  /* -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none; */
  /* Un padding-right extra para el icono de la flecha si usas un custom icon */
  /* padding-right: 2.5rem; */ 
}

/* Estilo para la barra de desplazamiento (Scrollbar) */
.custom-scrollbar::-webkit-scrollbar {
  width: 8px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: #a0aec0; /* gray-400 */
  border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: #718096; /* gray-500 */
}
</style>