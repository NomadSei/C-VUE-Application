<template>
  <div id="app" class="container">
    <h1 class="title">Tanques y Cuartos Disponibles</h1>

    <div class="message" v-if="loading">
      <div class="spinner"></div>
      Cargando datos...
    </div>
    <div class="message error" v-else-if="error">{{ error }}</div>

    <transition name="fade">
      <DataTable v-if="!loading && !error" :value="tanquesDisponibles" :paginator="true" :rows="10" :responsive="true" :filterDisplay="'row'">
        <Column field="id_tanque" header="ID Tanque"></Column>
        <Column field="descripcion_tanque" header="Descripción"></Column>
        <Column field="estatus_tanque" header="Estatus"></Column>
      </DataTable>
    </transition>

    <transition name="fade">
      <DataTable v-if="!loading && !error" :value="cuartosDisponibles" :paginator="true" :rows="10" :responsive="true" :filterDisplay="'row'">
        <Column field="id_cuarto_secado" header="ID Cuarto"></Column>
        <Column field="descripcion_cuarto" header="Descripción"></Column>
        <Column field="estatus_cuarto" header="Estatus"></Column>
      </DataTable>
    </transition>

    <div v-if="!loading && !error" class="chart-container">
      <canvas id="tanquesChart"></canvas>
    </div>

    <div v-if="!loading && !error" class="chart-container">
      <canvas id="cuartosChart"></canvas>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';
import { nextTick, onMounted, ref } from 'vue';

export default {
  setup() {
    const tanquesDisponibles = ref([]);  
    const cuartosDisponibles = ref([]);  
    const loading = ref(true);
    const error = ref(null);

    onMounted(() => {
      fetch('http://204.236.201.162:8080')
        .then(response => {
          if (!response.ok) {
            throw new Error('Error al cargar los datos: ' + response.statusText);
          }
          return response.json();
        })
        .then(data => {
          if (data && Array.isArray(data.tanquesDisponibles) && Array.isArray(data.cuartosDisponibles)) {
            tanquesDisponibles.value = data.tanquesDisponibles;  
            cuartosDisponibles.value = data.cuartosDisponibles;  
            loading.value = false;
            nextTick(() => {
              createCharts(); 
            });
          } else {
            throw new Error('La estructura de datos es incorrecta');
          }
        })
        .catch(err => {
          error.value = 'Error al cargar los datos: ' + err.message;
          loading.value = false;
        });
    });

    const createCharts = () => {
      const tanquesCanvas = document.getElementById('tanquesChart');
      const tanquesLabels = tanquesDisponibles.value.map(t => t.descripcion_tanque);
      const tanquesData = tanquesDisponibles.value.map(t => t.estatus_tanque === "Disponible" ? 1 : 0);

      new Chart(tanquesCanvas, {
        type: 'bar',
        data: {
          labels: tanquesLabels,
          datasets: [
            {
              label: 'Estatus de Tanques',
              data: tanquesData,
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              borderColor: 'rgba(75, 192, 192, 1)',
              borderWidth: 1,
            },
          ],
        },
        options: {
          responsive: true,
          animation: {
            duration: 1500,
            easing: 'easeOutBounce',
          },
          scales: {
            y: {
              beginAtZero: true,
            },
          },
        },
      });

      const cuartosCanvas = document.getElementById('cuartosChart');
      const cuartosLabels = cuartosDisponibles.value.map(c => c.descripcion_cuarto);
      const cuartosData = cuartosDisponibles.value.map(c => c.estatus_cuarto === "Disponible" ? 1 : 0);

      new Chart(cuartosCanvas, {
        type: 'bar',
        data: {
          labels: cuartosLabels,
          datasets: [
            {
              label: 'Estatus de Cuartos',
              data: cuartosData,
              backgroundColor: 'rgba(153, 102, 255, 0.2)',
              borderColor: 'rgba(153, 102, 255, 1)',
              borderWidth: 1,
            },
          ],
        },
        options: {
          responsive: true,
          animation: {
            duration: 1500,
            easing: 'easeOutBounce',
          },
          scales: {
            y: {
              beginAtZero: true,
            },
          },
        },
      });
    };

    return {
      tanquesDisponibles,
      cuartosDisponibles,
      loading,
      error,
    };
  },
};
</script>

<style scoped>

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Poppins', sans-serif;
  background-color: #f7f7f7;
  border-radius: 10px;
}

.title {
  text-align: center;
  font-size: 2.5em;
  color: #333;
  margin-bottom: 20px;
  text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.1);
}

.message {
  text-align: center;
  font-size: 1.2em;
  color: #555;
  padding: 10px;
  margin-bottom: 20px;
  background-color: #f4f4f4;
  border-radius: 5px;
}

.error {
  color: #e74c3c;
  background-color: #fce4e4;
}

.table-container {
  overflow-x: auto;
  margin-top: 20px;
  border-radius: 10px;
  padding: 10px;
  background-color: white;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.chart-container {
  text-align: center;
  margin-top: 20px;
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

canvas {
  max-width: 100%;
  border-radius: 10px;
}

.spinner {
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
  margin: 0 auto 10px auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}


.fade-enter-active, .fade-leave-active {
  transition: opacity 1s ease;
}
.fade-enter, .fade-leave-to 
{
  opacity: 0;
}

.error {
  color: #e74c3c;
  background-color: #fce4e4;
  animation: bounceIn 1s ease;
}

@keyframes bounceIn {
  0% {
    opacity: 0;
    transform: translateY(-200px);
  }
  60% {
    opacity: 1;
    transform: translateY(30px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

</style>
