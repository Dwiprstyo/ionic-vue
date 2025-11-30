<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Weather</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Weather</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="ion-padding">
        <h2>Cuaca Saat Ini di {{ location }}</h2>
        <div v-if="loading">Memuat data cuaca...</div>
        <div v-else-if="error">{{ error }}</div>
        <div v-else>
          <ion-card>
            <ion-card-header>
              <ion-card-title>Suhu Saat Ini</ion-card-title>
            </ion-card-header>
            <ion-card-content>
              <h1>{{ currentTemp }}°C</h1>
            </ion-card-content>
          </ion-card>
          <ion-card>
            <ion-card-header>
              <ion-card-title>Perkiraan Suhu Per Jam</ion-card-title>
            </ion-card-header>
            <ion-card-content>
              <ul>
                <li v-for="(hour, index) in hourlyTemps" :key="index">
                  {{ hour.time }}: {{ hour.temp }}°C
                </li>
              </ul>
            </ion-card-content>
          </ion-card>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonTitle, 
  IonContent, 
} from '@ionic/vue';

// Reactive variables
const currentTemp = ref<number>(0);
const location = ref<string>('Jakarta');
const hourlyTemps = ref<any[]>([]);
const loading = ref<boolean>(false);
const error = ref<string>('');

// Fetch weather data
const fetchWeatherData = async () => {
  loading.value = true;
  error.value = '';
  
  try {
    const response = await fetch('https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m');
    
    if (!response.ok) {
      throw new Error('Failed to fetch weather data');
    }
    
    const data = await response.json();
    
    currentTemp.value = Math.round(data.hourly.temperature_2m[0]);
    
    hourlyTemps.value = data.hourly.time.slice(0, 24).map((time: string, index: number) => ({
      time: new Date(time).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' }),
      temp: Math.round(data.hourly.temperature_2m[index])
    }));
    
  } catch (err) {
    error.value = 'Gagal memuat data cuaca';
    console.error('Error:', err);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchWeatherData();
});
</script>