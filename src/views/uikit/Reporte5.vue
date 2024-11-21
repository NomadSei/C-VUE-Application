<template>
  <div id="app" class="container">
    <h1 class="title">Reporte de Tipo de Café con Mayor Recolección Anual</h1>

    <div class="input-group">
      <label for="year">Año:</label>
      <input type="number" v-model="year" id="year" placeholder="Ingresa el año" />
      <button @click="loadData">Cargar Datos</button>
    </div>

    <div class="message" v-if="loading">
      <div class="spinner"></div>
      Cargando datos...
    </div>
    <div class="message error" v-else-if="error">{{ error }}</div>

    <transition name="fade">
      <DataTable v-if="!loading && !error" :value="reporteTipoCafeMayorAnual" :paginator="true" :rows="10" :responsive="true" :filterDisplay="'row'">
        <Column field="tipo_cafe" header="Tipo de Café"></Column>
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
    const reporteTipoCafeMayorAnual = ref([]);
    const loading = ref(true);
    const error = ref(null);
    const year = ref(2024);

    const loadData = () => {
      loading.value = true;
      error.value = null;
      fetch(`http://204.236.201.162:8080/?year=${year.value}`)
        .then(response => {
          if (!response.ok) {
            throw new Error('Error al cargar los datos: ' + response.statusText);
          }
          return response.json();
        })
        .then(data => {
          if (data && Array.isArray(data.reporteTipoCafeMayorAnual)) {
            reporteTipoCafeMayorAnual.value = data.reporteTipoCafeMayorAnual;
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
    };

    onMounted(loadData);

    const createChart = () => {
      const canvas = document.getElementById('myChart');
      if (!canvas) {
        console.error('El canvas no está disponible');
        return;
      }

      const labels = reporteTipoCafeMayorAnual.value.map(f => f.tipo_cafe);
      const totalKg = reporteTipoCafeMayorAnual.value.map(f => parseFloat(f.total_kg));

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
      reporteTipoCafeMayorAnual,
      loading,
      error,
      year,
      loadData,
    };
  },
};
</script>


<style scoped>

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 30px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f9fafb;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}


.title {
  text-align: center;
  font-size: 2.8em;
  color: #34495e;
  margin-bottom: 30px;
  font-weight: bold;
}


.message {
  text-align: center;
  font-size: 1.2em;
  color: #555;
  padding: 10px;
  margin-bottom: 20px;
  background-color: #f4f4f4;
  border-radius: 8px;
}

.error {
  color: #e74c3c;
  background-color: #fce4e4;
}


.input-group {
  display: flex;
  justify-content: center; 
  align-items: center; 
  gap: 10px;
  margin-bottom: 20px;
}

label {
  font-size: 1.2em; 
  font-weight: bold; 
  color: #333; 
  margin-right: 10px; 
  line-height: 1.6; 
}

input, button {
  padding: 12px 20px;
  font-size: 1.1em;
  border-radius: 8px;
  border: 1px solid #ccc;
  margin: 0;
  outline: none;
  transition: all 0.3s ease;
}

input:focus, button:hover {
  border-color: #3498db;
  box-shadow: 0 0 8px rgba(52, 152, 219, 0.6);
}

button {
  background-color: #3498db;
  color: white;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s ease;
  min-width: 120px;
}


button:hover {
  background-color: #2980b9;
}


.table-container, .chart-container {
  text-align: center;
  margin-top: 30px;
  background-color: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}


.spinner {
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
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
