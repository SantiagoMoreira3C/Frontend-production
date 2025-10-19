<template>
  <div class="p-8 bg-gray-50 min-h-screen">
    <h1 class="text-3xl font-bold mb-6 text-gray-800">Estadisticas Generales</h1>

    <div class="bg-white p-6 rounded-lg shadow-md mb-8">
      <h2 class="text-2xl font-semibold mb-4 text-gray-700">Filtros de estadistica general</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-2 gap-4">

        <div>
          <label class="block text-sm font-medium text-gray-600">Rango de Fechas</label>
          <div class="flex items-center space-x-2">
            <input type="date" v-model="filters.startDate" class="form-input" />
            <span>a</span>
            <input type="date" v-model="filters.endDate" class="form-input" />
          </div>
        </div>

        <div>
          <label for="career" class="block text-sm font-medium text-gray-600">Carrera</label>
          <select id="career" v-model="filters.career" class="form-select">
            <option value="">Todas</option>
            <option v-for="c in uniqueCareers" :key="c" :value="c">{{ c }}</option>
          </select>
        </div>

        <div>
          <label for="gender" class="block text-sm font-medium text-gray-600">Género</label>
          <select id="gender" v-model="filters.gender" class="form-select">
            <option value="">Todos</option>
            <option v-for="g in uniqueGenders" :key="g" :value="g">{{ g }}</option>
          </select>
        </div>

        <div>
          <label for="analysis-type" class="block text-sm font-medium text-gray-600">Tipo de Análisis de Riesgo</label>
          <select id="analysis-type" v-model="filters.analysisType" class="form-select">
            <option value="ideationRiskLevel">Nivel de Riesgo de Ideación</option>
            <option value="behaviorRiskLevel">Nivel de Riesgo de Comportamiento</option>
          </select>
        </div>

        <div>
          <label for="psychologist" class="block text-sm font-medium text-gray-600">Psicólogo Evaluador</label>
<select id="psychologist" v-bind:value="filters.psychologistId" @change="handlePsychologistChange" class="form-select">
        <option value="">Todos</option>
        <option 
            v-for="psy in psychologists" 
           :key="psy.uid"                 v-bind:value="psy.uid"  >
            {{ psy.firstName }} {{ psy.lastName }}
        </option>
    </select>
        </div>
      </div>

      <div v-if="!isLoading" class="mt-8 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        <!-- Gráfico de Distribución de Riesgo -->
        <div class="bg-white p-6 rounded-lg shadow-md col-span-full">
          <p class="text-sm text-gray-500 text-center mb-4">{{ displayedFilters }}</p>
          <div v-if="filteredAssessments.length > 0">
            <Bar :data="barChartData" :options="chartOptions" />
          </div>
          <div v-else class="text-center text-gray-500">
            <p>No hay datos de evaluaciones disponibles para este filtro.</p>
          </div>
        </div>
      </div>
      <!-- Sección de Gráficos -->

      <div v-else class="bg-white p-6 rounded-lg shadow-md min-h-[300px] text-center">
        <p class="text-lg text-gray-500">Cargando datos...</p>
      </div>
    </div>
    <div class="mt-4 flex justify-end">
      <button @click="applyFilters"
        class="px-4 py-2 bg-blue-600 text-white rounded-md shadow-sm hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500">
        Aplicar Filtros
      </button>
    </div>
  </div>

  <StaticRiskLevel/>
  <StaticLineEvaluation/>
  <StaticCareerRiskLevel/>
  <StaticsGenderRiskLevel/>
  <StaticsAgaRiskLevel/>
  <FrecuencyQuestionsStatics/>
  <FrecuencyQuestionsBehaviorStatics/>
</template>

<script>
const API_URL = import.meta.env.VITE_API_BASE_URL;
import FrecuencyQuestionsBehaviorStatics from '@/components/FrecuencyQuestionsBehaviorStatics.vue';
import FrecuencyQuestionsStatics from '@/components/FrecuencyQuestionsStatics.vue';
import StaticCareerRiskLevel from '@/components/StaticCareerRiskLevel.vue';
import StaticLineEvaluation from '@/components/StaticLineEvaluation.vue';
import StaticRiskLevel from '@/components/StaticRiskLevel.vue';
import StaticsAgaRiskLevel from '@/components/StaticsAgaRiskLevel.vue';
import StaticsGenderRiskLevel from '@/components/StaticsGenderRiskLevel.vue';
import axios from 'axios';
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js';
import { Bar } from 'vue-chartjs';

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale);

export default {
  components: { Bar, StaticRiskLevel, FrecuencyQuestionsBehaviorStatics, FrecuencyQuestionsStatics , StaticLineEvaluation, StaticCareerRiskLevel,StaticsGenderRiskLevel, StaticsAgaRiskLevel },
  data() {
    return {
      filters: {
        startDate: '',
        endDate: '',
        career: '',
        gender: '',
        psychologistId: '',
        analysisType: 'ideationRiskLevel'
      },
      isLoading: false,
      psychologists: [],
      studenslist: [],
      assessments: [],
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: {
            beginAtZero: true,
            title: {
              display: true,
              text: 'Número de Evaluaciones'
            }
          },
          x: {
            title: {
              display: true,
              text: 'Nivel de Riesgo'
            }
          }
        }
      }
    };
  },
  async mounted() {
    await this.fetchPsychologists();
    await this.fetchStudentDetails();
    this.fetchAssessments();
  },
  computed: {
    uniqueCareers() {
      const careers = this.studenslist.map(student => student.career).filter(Boolean);
      return [...new Set(careers)];
    },
    uniqueGenders() {
      const genders = this.studenslist.map(student => student.gender).filter(Boolean);
      return [...new Set(genders)];
    },
    displayedFilters() {
      const parts = [];
      if (this.filters.startDate && this.filters.endDate) {
        parts.push(`Fechas: ${this.filters.startDate} a ${this.filters.endDate}`);
      }
      if (this.filters.career) {
        parts.push(`Carrera: ${this.filters.career}`);
      }
      if (this.filters.gender) {
        parts.push(`Género: ${this.filters.gender}`);
      }
      if (this.filters.psychologistId) {
        const psychologist = this.psychologists.find(p => p._id === this.filters.psychologistId);
        if (psychologist) {
          parts.push(`Psicólogo: ${psychologist.firstName} ${psychologist.lastName}`);
        }
      }
      return parts.length ? `Filtros aplicados: ${parts.join(', ')}` : 'No hay filtros aplicados.';
    },
    filteredAssessments() {
      let startDate = null;
      let endDate = null;

      // --- Lógica de Manejo de Fechas (Se mantiene igual) ---
      if (this.filters.startDate) {
        const [year, month, day] = this.filters.startDate.split('-').map(Number);
        startDate = new Date(Date.UTC(year, month - 1, day, 0, 0, 0, 0)).getTime();
      }

      if (this.filters.endDate) {
        const [year, month, day] = this.filters.endDate.split('-').map(Number);
        endDate = new Date(Date.UTC(year, month - 1, day, 23, 59, 59, 999)).getTime();
      }

      if (!this.assessments || this.assessments.length === 0) {
        return [];
      }
      // --------------------------------------------------------

      const filtered = this.assessments.filter(assessment => {
        const student = assessment.studentInfo || this.studenslist.find(s => s._id === assessment.student);

        // 🔑 MODIFICADO: ELIMINADO el filtro local por psychologistId.
        // Asumimos que si this.filters.psychologistId tiene valor, se hizo una
        // nueva llamada al backend con ese filtro y 'this.assessments' ya está filtrado.
        // Si 'this.assessments' se obtiene de una llamada sin filtros de psicólogo, 
        // puedes usar el campo 'psychologistInfo._id' que proyectamos en el backend.

        // NOTA: Si los filtros de FECHA se mandan al backend, esta lógica también 
        // debería eliminarse o simplificarse. Aquí, mantenemos la lógica local de
        // carrera/género, ya que el backend no los maneja eficientemente.

        const careerMatch = !this.filters.career || (student && student.career === this.filters.career);
        const genderMatch = !this.filters.gender || (student && student.gender === this.filters.gender);

        const assessmentDateTimestamp = new Date(assessment.date).getTime();

        if (isNaN(assessmentDateTimestamp)) {
          return false;
        }

        // Si el filtro de FECHA es local:
        const dateMatch = (!startDate || assessmentDateTimestamp >= startDate) && (!endDate || assessmentDateTimestamp <= endDate);

        // 🔑 FILTRADO DE RETORNO SIMPLIFICADO: El filtro de psicólogo y, 
        // idealmente, el de fecha, ya los hace el backend.
        return careerMatch && genderMatch && dateMatch;
      });

      return filtered;
    },
    chartTitle() {
      const titleMap = {
        'ideationRiskLevel': 'Distribución de Nivel de Riesgo de Ideación',
        'behaviorRiskLevel': 'Distribución de Nivel de Riesgo de Comportamiento'
      };
      return titleMap[this.filters.analysisType] || 'Distribución de Nivel de Riesgo';
    },
    barChartData() {
      if (this.filteredAssessments.length === 0) {
        return { labels: [], datasets: [] };
      }

      const riskCounts = this.filteredAssessments.reduce((acc, assessment) => {
        const riskLevel = assessment[this.filters.analysisType] || 'Desconocido';
        acc[riskLevel] = (acc[riskLevel] || 0) + 1;
        return acc;
      }, {});

      const labels = Object.keys(riskCounts);
      const data = Object.values(riskCounts);

      const backgroundColors = labels.map(label => {
        switch (label) {
          case 'BAJO':
            return '#4299e1'; // Blue
          case 'MODERADO-BAJO':
            return '#63b3ed'; // Tailwind blue-400
          case 'MODERADO':
            return '#f6ad55'; // Tailwind orange-500
          case 'ALTO':
            return '#ed8936'; // Tailwind orange-600
          case 'MUY_ALTO':
            return '#f56565'; // Tailwind red-500
          case 'EXTREMO':
            return '#c53030'; // Tailwind red-700
          default:
            return '#a0aec0'; // Tailwind gray-500
        }
      });

      return {
        labels: labels,
        datasets: [{
          label: 'Número de Evaluaciones',
          backgroundColor: backgroundColors,
          data: data
        }]
      };
    }
  },
  methods: {
    async fetchPsychologists() {
      try {
        const token = localStorage.getItem('x-token');
        if (!token) {
          console.error('No se encontró el token de autenticación.');
          return;
        }
        const response = await axios.get(`${API_URL}/api/psicologist/getPsicologo`, {
          headers: {
            'x-token': token
          }
        });
        this.psychologists = response.data.psychologists;
      } catch (error) {
        console.error('Error al obtener la lista de psicólogos:', error);
      }
    },
    async fetchStudentDetails() {
      try {
        const token = localStorage.getItem('x-token');
        const response = await axios.get(`${API_URL}/api/students`, {
          headers: {
            'x-token': token
          }
        });
        this.studenslist = response.data.students;
      } catch (error) {
        console.error('Error al obtener detalles del estudiante:', error);
      }
    },

    async fetchAssessments() {
      this.isLoading = true;
      try {
        const token = localStorage.getItem('x-token');
        if (!token) {
          console.error('No se encontró el token de autenticación.');
          this.isLoading = false;
          return;
        }
const cleanedFilters = {};
for (const key in this.filters) {
    const value = this.filters[key];
    // 🔑 Esto asegura que si this.filters.psychologistId es "" NO se incluye.
    if (value !== '' && value !== null && value !== undefined && value !== 'undefined') { 
        cleanedFilters[key] = value;
    }
}

        console.log("[FRONTEND] Parámetros de consulta enviados (limpios):", cleanedFilters);
        // 🔑 CORRECCIÓN: Creamos un objeto de filtros limpios

        const response = await axios.get(`${API_URL}/api/statistics/all-assessments`, {
          params: cleanedFilters, // <-- ENVIAMOS LOS FILTROS LIMPIOS
          headers: {
            'x-token': token
          }
        });

        this.assessments = response.data;

      } catch (error) {
        console.error('Error al obtener datos de la API:', error.message);
        console.error('Detalles del error:', error.response || error);
      } finally {
        this.isLoading = false;
      }
    },

    // 🔑 NUEVO: Función para aplicar filtros y recargar datos (útil para el botón 'Aplicar')
    applyFilters() {
      this.fetchAssessments();
    },
    // En tu script de Vue.js (Sección methods)
handlePsychologistChange(event) {
    let incomingValue = event.target.value;
    
    // 1. Limpiamos cualquier valor nulo/no deseado a string ""
    if (incomingValue === undefined || incomingValue === null || incomingValue === 'undefined' || incomingValue === 'Todos') {
        incomingValue = '';
    }
    
    // 2. Si el valor contiene espacios, significa que el bindeo falló y nos envió el nombre.
    // Si el bindeo falló, no podemos saber el ID, así que lo forzamos a vacío.
    if (String(incomingValue).includes(' ')) {
        console.warn("[FRONTEND WARNING] Binding fallido: Recibido nombre en lugar de ID. Forzando a 'Todos'.");
        incomingValue = '';
    }
    
    // Asignamos el ID (UUID o "")
    this.filters.psychologistId = String(incomingValue);
    
    // El log ahora debería mostrar el UUID si el bindeo funciona.
    console.log(`[FRONTEND - CONTROL FINAL] ID para filtro: "${this.filters.psychologistId}"`); 
    
    // NO LLAMAMOS a fetchAssessments aquí.
}

  },

};
</script>

<style scoped>
.form-input,
.form-select {
  @apply w-full p-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500;
}
</style>
