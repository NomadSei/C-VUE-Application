<template>
  <div id="app" class="container">
    <h1 class="title">Selecciona un Reporte</h1>

    <div class="button-container">
      <button 
        v-for="i in 6" 
        :key="i" 
        @click="loadReport(i)" 
        :disabled="isLoading"
        :class="{ loading: isLoading }"
      >
        Reporte {{ i }}
      </button>
    </div>

    <div v-if="isLoading" class="spinner"></div>

    <transition name="fade" mode="out-in">
      <component 
        :is="`Reporte${activeReport}`" 
        v-if="!isLoading && reportLoaded" 
        class="report-content"
      />
    </transition>
  </div>
</template>

<script>
import Reporte1 from "@/views/uikit/Reporte1.vue";
import Reporte2 from "@/views/uikit/Reporte2.vue";
import Reporte3 from "@/views/uikit/Reporte3.vue";
import Reporte4 from "@/views/uikit/Reporte4.vue";
import Reporte5 from "@/views/uikit/Reporte5.vue";
import Reporte6 from "@/views/uikit/Reporte6.vue";
import { ref } from "vue";

export default {
  components: {
    Reporte1,
    Reporte2,
    Reporte3,
    Reporte4,
    Reporte5,
    Reporte6,
  },
  setup() {
    const activeReport = ref(0);
    const reportLoaded = ref(false);
    const isLoading = ref(false);

    const loadReport = (reportNumber) => {
      activeReport.value = reportNumber;
      reportLoaded.value = false;
      isLoading.value = true;

      setTimeout(() => {
        reportLoaded.value = true;
        isLoading.value = false;
      }, 1000);
    };

    return { activeReport, loadReport, reportLoaded, isLoading };
  },
};
</script>

<style scoped>

.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: "Poppins", "Roboto", Arial, sans-serif; 
  background: linear-gradient(135deg, white, #d9e4f5);
  border-radius: 15px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.title {
  font-size: 2.5em;
  color: #333;
  font-family: "Poppins", sans-serif; 
  font-weight: 600;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
  margin-bottom: 20px;
}

.button-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 20px;
  margin: 20px 0;
  width: 100%;
}

button {
  padding: 15px;
  font-size: 1em;
  font-family: "Roboto", sans-serif; 
  font-weight: bold;
  color: #fff;
  background: linear-gradient(45deg, #007bff, #0056b3);
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s, background 0.3s;
}

button:hover {
  box-shadow: 0 4px 10px rgba(0, 123, 255, 0.5);
  background: linear-gradient(45deg, #0056b3, #003f7f);
}

button:active {
  transform: scale(0.95);
}

button.loading {
  opacity: 0.7;
  cursor: not-allowed;
}

.spinner {
  margin: 20px auto;
  border: 6px solid #f3f3f3;
  border-top: 6px solid #007bff;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.report-content {
  margin-top: 20px;
  width: 100%;
  text-align: center;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter, 
.fade-leave-to {
  opacity: 0;
}
</style>
