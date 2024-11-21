<template>
  <div id="app" class="container">
    <h1 class="title">Duración Total de los Lotes</h1>

    <div class="message" v-if="loading">
      <div class="spinner"></div>
      Cargando datos...
    </div>
    <div class="message error" v-else-if="error">{{ error }}</div>

    <transition name="fade">
      <DataTable v-if="!loading && !error" :value="duracionProceso" :paginator="true" :rows="10" :responsive="true" :filterDisplay="'row'">
        <Column field="id_lote" header="ID Lote"></Column>
        <Column field="fase" header="Fase"></Column>
        <Column field="duracion_total" header="Duración Total"></Column>
        <Column field="total_horas" header="Total Horas"></Column>
        <Column field="total_dias" header="Total Días"></Column>
        <Column field="resto_horas" header="Resto Horas"></Column>
      </DataTable>
    </transition>

    <div v-if="!loading && !error" class="chart-container">
      <canvas id="myChart"></canvas>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';
import { nextTick, onMounted, ref } from 'vue';

export default {
  setup() {
    const duracionProceso = ref([]);
    const loading = ref(true);
    const error = ref(null);

    onMounted(() => {
      // Cambiar la URL para usar el proxy en Vercel
      fetch('http://204.236.201.162:8080') // Llamada al endpoint de la función serverless
        .then(response => {
          if (!response.ok) {
            throw new Error('Error al cargar los datos: ' + response.statusText);
          }
          return response.json();
        })
        .then(data => {
          if (data && Array.isArray(data.duracionProceso)) {
            duracionProceso.value = data.duracionProceso;
            loading.value = false;
            nextTick(() => { 
              createChart();
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

    const createChart = () => {
      const canvas = document.getElementById('myChart');
      if (!canvas) {
        console.error('El canvas no está disponible');
        return;
      }

      const labels = duracionProceso.value.map(d => d.id_lote);
      const totalHoras = duracionProceso.value.map(d => parseFloat(d.total_horas));

      new Chart(canvas, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [
            {
              label: 'Total Horas',
              data: totalHoras,
              backgroundColor: 'rgba(255, 99, 132, 0.2)',
              borderColor: 'rgba(255, 99, 132, 1)',
              borderWidth: 1,
            }
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
      duracionProceso,
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
  font-family: 'Poppins', 'Roboto', sans-serif;
  background-color: #f7f7f7;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  font-size: 2.5em;
  color: #333;
  margin-bottom: 20px;
  text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.1);
  font-weight: 600;
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
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.chart-container {
  text-align: center;
  margin-top: 20px;
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

canvas {
  max-width: 100%;
  border-radius: 10px;
}

/* Estilos del spinner */
.spinner {
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
  margin: 0 auto 10px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 1s ease;
}

.fade-enter, .fade-leave-to {
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
