<template>
  <div id="app" class="container">
    <h1 class="title">Fase Actual de Lotes</h1>

    <div class="message" v-if="loading">
      <div class="spinner"></div>
      Cargando datos...
    </div>
    <div class="message error" v-else-if="error">{{ error }}</div>

    <transition name="fade">
      <DataTable v-if="!loading && !error" :value="faseActualSimple" :paginator="true" :rows="10" :responsive="true" :filterDisplay="'row'">
        <Column field="id_lote" header="ID Lote"></Column>
        <Column field="fase" header="Fase"></Column>
        <Column field="kg_cafe" header="Kg de Café"></Column>
        <Column field="tipo_cafe" header="Tipo de Café"></Column>
        <Column field="fecha_inicio" header="Fecha de Inicio"></Column>
        <Column field="estatus" header="Estatus"></Column>
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
    const faseActualSimple = ref([]);
    const loading = ref(true);
    const error = ref(null);

    onMounted(() => {
      // Cambiar la URL para usar el proxy en Vercel
      fetch('http://204.236.201.162:8080') // Llamada al endpoint del proxy
        .then(response => {
          if (!response.ok) {
            throw new Error('Error al cargar los datos: ' + response.statusText);
          }
          return response.json();
        })
        .then(data => {
          if (data && Array.isArray(data.faseActualSimple)) {
            faseActualSimple.value = data.faseActualSimple;
            loading.value = false;
            console.log("Datos cargados:", data.faseActualSimple);
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

      const labels = faseActualSimple.value.map(f => f.id_lote);
      const kgCafe = faseActualSimple.value.map(f => parseFloat(f.kg_cafe));

      new Chart(canvas, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [
            {
              label: 'Kg de Café',
              data: kgCafe,
              backgroundColor: 'rgba(255, 99, 132, 0.2)',
              borderColor: 'rgba(255, 99, 132, 1)',
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
      faseActualSimple,
      loading,
      error,
    };
  },
};
</script>

<style scoped>

body {
  font-family: 'Poppins', 'Roboto', sans-serif;
  margin: 0;
  padding: 0;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 30px;
  background-color: #f7f7f7;
  border-radius: 10px;
}

.title {
  text-align: center;
  font-size: 2.8em;
  color: #333;
  margin-bottom: 30px;
  text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.1);
}

.message {
  text-align: center;
  font-size: 1.3em;
  color: #555;
  padding: 15px;
  margin-bottom: 30px;
  background-color: #f4f4f4;
  border-radius: 5px;
}

.error {
  color: #e74c3c;
  background-color: #fce4e4;
}

.table-container {
  overflow-x: auto;
  margin-top: 30px;
  padding: 15px;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.chart-container {
  text-align: center;
  margin-top: 30px;
  background-color: white;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
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
  width: 35px;
  height: 35px;
  animation: spin 1s linear infinite;
  margin: 0 auto 15px auto;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
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
