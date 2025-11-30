<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Crypto</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Crypto</ion-title>
        </ion-toolbar>
      </ion-header>

      <!-- Loading State -->
      <div v-if="loading" class="loading-container">
        <ion-spinner></ion-spinner>
        <p>Memuat data cryptocurrency...</p>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="error-container">
        <p>{{ error }}</p>
        <ion-button @click="fetchCryptoData()" color="primary">Coba Lagi</ion-button>
      </div>

      <!-- Crypto Data -->
      <div v-else class="crypto-container">
        <!-- Refresh Button -->
        <div class="refresh-container">
          <ion-button 
            expand="block" 
            @click="fetchCryptoData()" 
            :disabled="loading"
            color="primary"
          >
            Refresh
          </ion-button>
        </div>

        <!-- Crypto Table -->
        <div class="crypto-table">
          <div 
            v-for="crypto in cryptoList" 
            :key="crypto.id" 
            class="crypto-row"
          >
            <div class="crypto-cell rank-cell">
              <div class="label">Rank</div>
              <div class="value">{{ crypto.rank }}</div>
            </div>
            <div class="crypto-cell name-cell">
              <div class="label">{{ crypto.name }}</div>
              <div class="value symbol">{{ crypto.symbol }}</div>
            </div>
            <div class="crypto-cell price-cell">
              <div class="label">USD</div>
              <div class="value price">{{ formatPrice(crypto.price_usd) }}</div>
            </div>
          </div>
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
  IonSpinner, 
  IonButton 
} from '@ionic/vue';

// Interface untuk data crypto
interface CryptoData {
  id: string;
  symbol: string;
  name: string;
  rank: number;
  price_usd: string;
}

// Reactive variables
const cryptoList = ref<CryptoData[]>([]);
const loading = ref<boolean>(false);
const error = ref<string>('');

// Fetch crypto data
const fetchCryptoData = async () => {
  loading.value = true;
  error.value = '';
  
  try {
    const response = await fetch('https://api.coinlore.net/api/tickers/');
    
    if (!response.ok) {
      throw new Error('Failed to fetch crypto data');
    }
    
    const data = await response.json();
    
    // Ambil top 7 cryptocurrency sesuai gambar
    cryptoList.value = data.data.slice(0, 20);
    
  } catch (err) {
    error.value = 'Gagal memuat data cryptocurrency';
    console.error('Error:', err);
  } finally {
    loading.value = false;
  }
};

// Format price
const formatPrice = (price: string): string => {
  const num = parseFloat(price);
  if (num >= 1) {
    return num.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
  } else {
    return num.toLocaleString('en-US', { minimumFractionDigits: 6, maximumFractionDigits: 6 });
  }
};

// Call function saat component dimount
onMounted(() => {
  fetchCryptoData();
});
</script>

<style scoped>
.loading-container, .error-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 200px;
  text-align: center;
  padding: 20px;
}

.crypto-container {
  padding: 16px;
}

.refresh-container {
  margin-bottom: 20px;
}

.crypto-table {
  display: flex;
  flex-direction: column;
  gap: 1px;
  background-color: #ccc;
  border-radius: 8px;
  overflow: hidden;
}

.crypto-row {
  display: flex;
  background-color: #f5f5dc;
  min-height: 60px;
}

.crypto-cell {
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 8px 12px;
  border-right: 1px solid #ccc;
}

.crypto-cell:last-child {
  border-right: none;
}

.rank-cell {
  flex: 0 0 80px;
  text-align: center;
}

.name-cell {
  flex: 1;
}

.price-cell {
  flex: 0 0 120px;
  text-align: right;
}

.label {
  font-size: 0.8rem;
  color: #666;
  margin-bottom: 2px;
  font-weight: 500;
}

.value {
  font-size: 1rem;
  font-weight: bold;
  color: #333;
}

.symbol {
  font-size: 1.1rem;
  color: #000;
}

.price {
  color: #000;
  font-family: monospace;
}

/* Responsive adjustments */
@media (max-width: 480px) {
  .rank-cell {
    flex: 0 0 60px;
  }
  
  .price-cell {
    flex: 0 0 100px;
  }
  
  .label {
    font-size: 0.7rem;
  }
  
  .value {
    font-size: 0.9rem;
  }
}
</style>
