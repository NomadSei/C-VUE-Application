<template>
  <div id="app" class="container">
    <h1 class="title">Reporte del Mes con Mayor Recolección</h1>

    <div class="message" v-if="loading">
      <div class="spinner"></div>
      Cargando datos...
    </div>

    <div class="message error" v-else-if="error">{{ error }}</div>

    <transition name="fade">
      <DataTable v-if="!loading && !error" :value="reporteMesMayorRecoleccion" :paginator="true" :rows="10" :responsive="true" :filterDisplay="'row'">
        <Column field="mes" header="Mes"></Column>
        <Column field="total_kg" header="Total de Kg"></Column>
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
    const reporteMesMayorRecoleccion = ref([]);
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
          if (data && Array.isArray(data.reporteMesMayorRecoleccion)) {
            reporteMesMayorRecoleccion.value = data.reporteMesMayorRecoleccion;
            loading.value = false;
            nextTick(() => {
              createChart(); // Llamar a la función para crear el gráfico
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
      if (!canvas) return;

      const labels = reporteMesMayorRecoleccion.value.map(f => f.mes);
      const totalKg = reporteMesMayorRecoleccion.value.map(f => parseFloat(f.total_kg));

      new Chart(canvas, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [
            {
              label: 'Total Kg de Café',
              data: totalKg,
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
    };

    return {
      reporteMesMayorRecoleccion,
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
  padding: 40px;
  font-family: 'Poppins', sans-serif;
  background-color: #f7f7f7;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  font-size: 2.5em;
  color: #333;
  margin-bottom: 30px;
  text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);
}

.message {
  text-align: center;
  font-size: 1.2em;
  padding: 15px;
  margin-bottom: 20px;
  background-color: #f4f4f4;
  border-radius: 10px;
  animation: fadeIn 1s ease-in;
}

.message.error {
  color: #e74c3c;
  background-color: #fce4e4;
}

.table-container {
  overflow-x: auto;
  margin-top: 30px;
  border-radius: 10px;
  padding: 15px;
  background-color: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.chart-container {
  margin-top: 40px;
  background-color: white;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

canvas {
  width: 100%;
  border-radius: 10px;
}

.spinner {
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin: 0 auto 10px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@keyframes fadeIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}

/* Transición de desvanecimiento */
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s ease;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}

/* Animación del mensaje de error */
.error {
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
