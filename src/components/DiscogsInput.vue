  <template>
    <div>
      <div class="bg-white rounded-lg shadow-md p-6 mb-4">
        <h2 class="text-lg font-semibold mb-4">Aplikasi Input Data Discogs</h2>
        <div class="tabs">
          <button :class="{ active: activeTab === 'input' }" @click="setActiveTab('input')">Input Data</button>
          <button :class="{ active: activeTab === 'view' }" @click="setActiveTab('view')">
            Lihat Data
            <span v-if="newRecordCount > 0" class="badge">{{ newRecordCount }}</span>
          </button>
        </div>
      </div>
  
      <div v-if="activeTab === 'input'" class="bg-white rounded-lg shadow-md p-6">
        <div v-if="notification.show && notification.type === 'success'" class="notification success">{{ notification.message }}</div>
        <div v-if="notification.show && notification.type === 'error'" class="notification error">{{ notification.message }}</div>
        <h2 class="text-lg font-semibold mb-4">Masukkan ID Album Discogs</h2>
        <div class="mb-4">
          <label for="discogsCode" class="block text-gray-700 text-sm font-bold mb-2">Kode Discogs:</label>
          <label for="discogsCode" class="block text-gray-700 text-sm font-bold mb-2">
            Contoh, Salin kode yang di Highlight ke form di bawah:
            <a href="https://www.discogs.com/release/" class="text-gray-400">https://www.discogs.com/release/</a>
            <span class="font-bold text-black bg-green-300">221824</span>
            <span class="text-gray-400">-Rick-Astley-Never-Gonna-Give-You-Up</span>
          </label>
          <input
            type="text"
            id="discogsCode"
            v-model="discogsCode"
            class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            placeholder="Contoh: 221824"
          />
        </div>
        <button
          @click="fetchReleaseInfo"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
        >
          Cari
        </button>
  
        <div v-if="releaseInfo" class="mt-4">
          <h3>Preview Data</h3>
          <p class="text-black"><strong>CATALOG NO:</strong> {{ releaseInfo.catalog_number }}</p>
          <p class="text-black"><strong>ARTIST:</strong> {{ releaseInfo.artist }}</p>
          <p class="text-black"><strong>TITLE:</strong> {{ releaseInfo.title }}</p>
          <p class="text-black"><strong>GENRE:</strong> {{ releaseInfo.genre ? releaseInfo.genre.join(', ') : '' }}</p>
          <p class="text-black"><strong>FORMAT:</strong> {{ releaseInfo.format ? releaseInfo.format.join(', ') : '' }}</p>
          <p class="text-black"><strong>LABEL:</strong> {{ releaseInfo.label ? releaseInfo.label.join(', ') : '' }}</p>
          <p class="text-black"><strong>RELEASED:</strong> {{ releaseInfo.released_year }}</p>
  
          <!-- <div class="mt-4">
            <label for="currency" class="block text-gray-700 text-sm font-bold mb-2">Mata Uang:</label>
            <select
              id="currency"
              v-model="selectedCurrency"
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            >
              <option value="USD">Dollar (USD)</option>
              <option value="EUR">Euro (EUR)</option>
            </select>
          </div>
  
          <div class="mt-4">
            <label :for="selectedCurrency === 'USD' ? 'priceUSD' : 'priceEUR'" class="block text-gray-700 text-sm font-bold mb-2">Harga {{ selectedCurrency === 'USD' ? 'Dollar' : 'Euro' }}:</label>
            <input
              type="number"
              :id="selectedCurrency === 'USD' ? 'priceUSD' : 'priceEUR'"
              v-model="currencyPrice"
              step="0.01"
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            />
          </div>
  
          <div v-if="convertedPriceIDR !== null" class="mt-4">
            <label for="priceIDR" class="block text-gray-700 text-sm font-bold mb-2">Harga (IDR):</label>
            <input
              type="text"
              id="priceIDR"
              :value="formatCurrency(convertedPriceIDR)"
              readonly
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100"
            />
          </div>
  
          <div class="mt-4">
            <label for="coverQuality" class="block text-gray-700 text-sm font-bold mb-2">Kualitas Cover (0.5 - 1):</label>
            <input
              type="number"
              id="coverQuality"
              v-model.number="coverQuality"
              step="0.1"
              min="0.5"
              max="1"
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            />
          </div>
  
          <div v-if="calculatedPrice !== null" class="mt-4">
            <label for="calculatedPrice" class="block text-gray-700 text-sm font-bold mb-2">Hasil Kalkulasi:</label>
            <input
              type="text"
              id="calculatedPrice"
              :value="formatCurrency(calculatedPrice)"
              readonly
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100"
            />
          </div>
  
          <div v-if="finalPrice !== null" class="mt-4">
            <label for="finalPrice" class="block text-gray-700 text-sm font-bold mb-2">Harga Final:</label>
            <input
              type="text"
              id="finalPrice"
              :value="formatCurrency(finalPrice)"
              readonly
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100"
            />
          </div> -->

<!-- <div class="mt-4">
  <div>
    <label>
      <input type="radio" v-model="hargaMode" value="otomatis" class="text-gray-700"> Harga Otomatis
    </label>
  </div>

  <div v-if="hargaMode === 'otomatis'" class="mt-2">
    <label for="currency" class="block text-gray-700 text-sm font-bold mb-2">Pilih Mata Uang:</label>
    <select
      id="currency"
      v-model="selectedCurrency"
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
    >
      <option value="USD">Dollar (USD)</option>
      <option value="EUR">Euro (EUR)</option>
    </select>

    <label :for="selectedCurrency === 'USD' ? 'priceUSD' : 'priceEUR'" class="block text-gray-700 text-sm font-bold mb-2">Harga {{ selectedCurrency === 'USD' ? 'Dollar' : 'Euro' }}:</label>
    <input
      type="number"
      :id="selectedCurrency === 'USD' ? 'priceUSD' : 'priceEUR'"
      v-model="currencyPrice"
      step="0.01"
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
    />

    <label for="priceIDR" class="block text-gray-700 text-sm font-bold mb-2">Harga (IDR):</label>
    <input
      type="text"
      id="priceIDR"
      :value="formatCurrency(convertedPriceIDR)"
      readonly
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100"
    />

    <label for="kualitasCoverInput" class="block text-gray-700 text-sm font-bold mb-2">Kualitas Cover (1-100):</label>
    <input
      type="number"
      id="kualitasCoverInput"
      v-model.number="kualitasCoverInput"
      min="1"
      max="100"
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
    />

    <label for="finalPriceOtomatis" class="block text-gray-700 text-sm font-bold mb-2">Harga Akhir:</label>
    <input
      type="text"
      id="finalPriceOtomatis"
      :value="formatCurrency(finalPrice)"
      readonly
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100"
    />
  </div>

  <div class="mt-4">
    <label>
      <input type="radio" v-model="hargaMode" value="manual" class="text-gray-700"> Add Harga Manual
    </label>
  </div>

  <div v-if="hargaMode === 'manual'" class="mt-2">
    <label for="hargaManual" class="block text-gray-700 text-sm font-bold mb-2">Masukkan Harga Manual (IDR):</label>
    <input
      type="number"
      id="hargaManual"
      v-model.number="hargaManual"
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
    />

    <label for="finalPriceManual" class="block text-gray-700 text-sm font-bold mb-2">Harga Akhir:</label>
    <input
      type="text"
      id="finalPriceManual"
      :value="formatCurrency(hargaManual)"
      readonly
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100"
    />
  </div>
</div> -->

<div class="mt-4 grid grid-cols-1 md:grid-cols-2 gap-4">
  <div class="border rounded p-4" :class="{ 'bg-gray-100': hargaMode === 'manual' }">
    <label class="block font-bold mb-2">
      <input type="radio" v-model="hargaMode" value="otomatis" class="mr-2"> Harga Otomatis
    </label>
    <div v-if="hargaMode === 'otomatis' || hargaMode === 'manual'">
      <label for="currency" class="block text-gray-700 text-sm font-semibold mb-1">Pilih Mata Uang:</label>
      <select
        id="currency"
        v-model="selectedCurrency"
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm"
        :disabled="hargaMode === 'manual'"
      >
        <option value="USD">Dollar (USD)</option>
        <option value="EUR">Euro (EUR)</option>
      </select>

      <label :for="selectedCurrency === 'USD' ? 'priceUSD' : 'priceEUR'" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Harga {{ selectedCurrency === 'USD' ? 'Dollar' : 'Euro' }}:</label>
      <div class="flex">
        <input
          type="number"
          :id="selectedCurrency === 'USD' ? 'priceUSD' : 'priceEUR'"
          v-model="currencyPrice"
          step="0.01"
          class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm"
          :disabled="hargaMode === 'manual'"
        />
        <span class="inline-flex items-center px-3 rounded border border-l-0 bg-gray-50 text-gray-500 text-sm">
          {{ selectedCurrency === 'USD' ? '$' : '€' }}
        </span>
      </div>

      <label for="priceIDR" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Setelah Konversi ke IDR:</label>
      <input
        type="text"
        id="priceIDR"
        :value="formatCurrency(convertedPriceIDR)"
        readonly
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100 text-sm"
        :disabled="hargaMode === 'manual'"
      />

      <label for="kualitasCoverInput" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Kualitas Cover (1-100):</label>
      <div class="flex">
        <input
          type="number"
          id="kualitasCoverInput"
          v-model.number="kualitasCoverInput"
          min="1"
          max="100"
          class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm"
          :disabled="hargaMode === 'manual'"
        />
        <span class="inline-flex items-center px-3 rounded border border-l-0 bg-gray-50 text-gray-500 text-sm">
          %
        </span>
      </div>

      <label for="finalPriceOtomatis" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Harga Akhir:</label>
      <input
        type="text"
        id="finalPriceOtomatis"
        :value="formatCurrency(finalPrice)"
        readonly
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100 text-sm"
        :disabled="hargaMode === 'manual'"
      />
    </div>
  </div>

  <div class="border rounded p-4" :class="{ 'bg-gray-100': hargaMode === 'otomatis' }">
    <label class="block font-bold mb-2">
      <input type="radio" v-model="hargaMode" value="manual" class="mr-2"> Add Harga Manual
    </label>
    <div v-if="hargaMode === 'manual' || hargaMode === 'otomatis'">
      <label for="hargaManual" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Masukkan Harga Manual (IDR):</label>
      <div class="flex">
        <input
          type="number"
          id="hargaManual"
          v-model.number="hargaManual"
          class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm"
          :disabled="hargaMode === 'otomatis'"
        />
        <span class="inline-flex items-center px-3 rounded border border-l-0 bg-gray-50 text-gray-500 text-sm">
          Rp
        </span>
      </div>

      <label for="finalPriceManual" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Harga Akhir:</label>
      <input
        type="text"
        id="finalPriceManual"
        :value="formatCurrency(finalPrice)"
        readonly
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline bg-gray-100 text-sm"
        :disabled="hargaMode === 'otomatis'"
      />
    </div>
  </div>
</div>
  
          <button
            @click="saveToIndexedDBWithCalculation"
            :disabled="!currencyPrice || !coverQuality"
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline mt-4"
          >
            Simpan
          </button>
        </div>
        <p v-else-if="fetchError" class="mt-4 text-red-500">{{ fetchError }}</p>
        <p v-else class="mt-4 text-gray-500 italic">Masukkan kode Discogs dan klik "Cari".</p>
      </div>
  
      <div v-if="activeTab === 'view'" class="bg-white rounded-lg shadow-md p-6 mt-4">
        <div v-if="notification.show && notification.type === 'success'" class="notification success">{{ notification.message }}</div>
        <div v-if="notification.show && notification.type === 'error'" class="notification error">{{ notification.message }}</div>
        <h2 class="text-lg font-semibold mb-4">Lihat Data Tersimpan</h2>
        <table v-if="storedData.length > 0" class="w-full border-collapse">
          <thead>
            <tr class="bg-yellow-400">
              <th class="border border-blue-300 p-2 text-left">No</th>
              <th class="border border-gray-300 p-2 text-left">Catalog No</th>
              <th class="border border-gray-300 p-2 text-left">Artist</th>
              <th class="border border-gray-300 p-2 text-left">Title</th>
              <th class="border border-gray-300 p-2 text-left">Genre</th>
              <th class="border border-gray-300 p-2 text-left">Format</th>
              <th class="border border-gray-300 p-2 text-left">Label</th>
              <th class="border border-gray-300 p-2 text-left">Released</th>
              <th class="border border-gray-300 p-2 text-left">Harga Asli</th>
              <th class="border border-gray-300 p-2 text-left">Harga Toko</th>
              <th class="border border-gray-300 p-2 text-left">Harga Discogs</th>
              <th class="border border-gray-300 p-2 text-left">Kualitas Cover</th>
              <th class="border border-gray-300 p-2 text-left">Hasil Kalkulasi</th>
              <th class="border border-gray-300 p-2 text-left">Harga Final</th>
              <th class="border border-gray-300 p-2 text-left">Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in storedData" :key="item.id">
              <td class="border border-gray-300 p-2 text-black">{{ index + 1 }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ item.data['CATALOG NO'] }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ item.data.ARTIST }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ item.data.TITLE }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ item.data.GENRE }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ item.data.FORMAT }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ item.data.LABEL }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ item.data.RELEASED }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ formatCurrency(item.data.PRICE_ORIGINAL) }} {{ item.data.CURRENCY }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ formatCurrency(item.data.PRICE_TOKO) }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ formatCurrency(item.data.PRICE_DISCOGS) }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ item.data.COVER_QUALITY }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ formatCurrency(item.data.CALCULATED_PRICE) }}</td>
              <td class="border border-gray-300 p-2 text-black">{{ formatCurrency(item.data.FINAL_PRICE) }}</td>
              <td class="border border-gray-300 p-2">
                <button @click="confirmDelete(item.id)" class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-2 rounded focus:outline-none focus:shadow-outline">Hapus</button>
              </td>
            </tr>
          </tbody>
        </table>
        <p v-else class="text-gray-500 italic">Tidak ada data tersimpan.</p>
  
        <button
          @click="exportToCsv"
          :disabled="storedData.length === 0"
          class="bg-indigo-500 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline mt-4 mr-2"
        >
          Ekspor ke CSV
        </button>
        <button
          @click="clearIndexedDB"
          :disabled="storedData.length === 0"
          class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline mt-4"
        >
          Kosongkan Data
        </button>
      </div>
    </div>
  </template>
  

<script>
export default {
  data() {
    return {
      hargaMode: 'otomatis', // Bisa 'otomatis' atau 'manual'
      hargaManual: null,
      kualitasCoverInput: null, // Untuk input skala 1-100
      activeTab: 'input',
      db: null,
      objectStoreName: 'discogsReleases',
      discogsCode: '',
      releaseInfo: null,
      fetchError: null,
      selectedCurrency: 'USD',
      currencyPrice: null,
      dollarToRupiahRate: null,
      euroToRupiahRate: null,
      convertedPriceIDR: null,
      coverQuality: null,
      calculatedPrice: null,
      finalPrice: null,
      storedData: [],
      notification: {
        show: false,
        type: '',
        message: '',
      },
      newRecordCount: 0,
    };
  },
  async mounted() {
    await this.initDatabase();
    await this.loadStoredData();
    await this.fetchExchangeRates();
  },
  watch: {
    currencyPrice() {
      this.convertCurrency();
      this.calculatePrices();
    },
    selectedCurrency() {
      this.currencyPrice = null;
      this.convertedPriceIDR = null;
      this.calculatePrices();
    },
    coverQuality() {
      this.calculatePrices();
    },
    releaseInfo(newVal) {
      if (newVal && newVal.format && newVal.format.includes('LP')) {
        this.hargaTokoOtomatis = 150000;
      } else {
        this.hargaTokoOtomatis = 100000;
      }
      this.calculatePrices();
    },
  },
  computed: {
    hargaTokoOtomatis() {
      if (this.releaseInfo && this.releaseInfo.format) {
        return this.releaseInfo.format.includes('LP') ? 150000 : 100000;
      }
      return 0;
    },
  },
  methods: {
    setActiveTab(tabName) {
      this.activeTab = tabName;
      console.log('activeTab:', this.activeTab);
      if (tabName === 'view') {
        this.newRecordCount = 0;
      }
    },
    async initDatabase() {
      return new Promise((resolve, reject) => {
        const request = indexedDB.open('discogsDB', 1);

        request.onupgradeneeded = (event) => {
          const db = event.target.result;
          if (!db.objectStoreNames.contains(this.objectStoreName)) {
            db.createObjectStore(this.objectStoreName, { keyPath: 'id', autoIncrement: true });
          }
        };

        request.onsuccess = (event) => {
          this.db = event.target.result;
          resolve();
        };

        request.onerror = (event) => {
          console.error('Gagal membuka database:', event.target.error);
          reject(event.target.error);
        };
      });
    },
    async loadStoredData() {
      return new Promise((resolve, reject) => {
        const transaction = this.db.transaction(this.objectStoreName, 'readonly');
        const objectStore = transaction.objectStore(this.objectStoreName);
        const getAllRequest = objectStore.getAll();

        getAllRequest.onsuccess = () => {
          this.storedData = getAllRequest.result.map(item => ({
            ...item,
            data: {
              ...item.data,
              GENRE: item.data.GENRE ? item.data.GENRE : [],
              FORMAT: item.data.FORMAT ? item.data.FORMAT : [],
              LABEL: item.data.LABEL ? item.data.LABEL : [],
            },
          }));
          resolve();
        };

        getAllRequest.onerror = (event) => {
          console.error('Gagal mengambil data tersimpan:', event.target.error);
          reject(event.target.error);
        };
      });
    },
    async fetchReleaseInfo() {
      this.releaseInfo = null;
      this.fetchError = null;
      this.clearNotification();
      try {
        const response = await fetch(`https://bhang-records-simplify-api.vercel.app/discogs/release_info/${this.discogsCode}`);
        if (!response.ok) {
          const message = `Terjadi kesalahan: ${response.status}`;
          throw new Error(message);
        }
        const data = await response.json();
        this.releaseInfo = data;
      } catch (error) {
        this.fetchError = error.message;
        this.showNotification('error', error.message);
      }
    },
    async fetchExchangeRates() {
  this.dollarToRupiahRate = 16000; // Hardcode nilai tukar USD ke IDR
  this.euroToRupiahRate = 17500;   // Hardcode nilai tukar EUR ke IDR

  console.log('Nilai Tukar USD ke IDR (Hardcoded):', this.dollarToRupiahRate);
  console.log('Nilai Tukar EUR ke IDR (Hardcoded):', this.euroToRupiahRate);
},
    convertCurrency() {
      if (this.currencyPrice !== null) {
        if (this.selectedCurrency === 'USD' && this.dollarToRupiahRate !== null) {
          this.convertedPriceIDR = this.currencyPrice * this.dollarToRupiahRate;
        } else if (this.selectedCurrency === 'EUR' && this.euroToRupiahRate !== null) {
          this.convertedPriceIDR = this.currencyPrice * this.euroToRupiahRate;
        } else {
          this.convertedPriceIDR = null;
        }
      } else {
        this.convertedPriceIDR = null;
      }
    },
    calculatePrices() {
      if (this.convertedPriceIDR !== null && this.coverQuality !== null) {
        const hargaToko = this.hargaTokoOtomatis;
        const hargaDiscogs = this.convertedPriceIDR;

        let hargaTengah = hargaToko;
        if (hargaDiscogs > hargaToko) {
          hargaTengah = hargaToko + (hargaDiscogs - hargaToko) / 2;
        }

        this.calculatedPrice = hargaTengah * this.coverQuality;
        this.finalPrice = Math.round(this.calculatedPrice / 5000) * 5000;
      } else {
        this.calculatedPrice = null;
        this.finalPrice = null;
      }
    },
    async saveToIndexedDBWithCalculation() {
      if (this.releaseInfo && this.convertedPriceIDR !== null && this.coverQuality !== null) {
        const recordToSave = {
          'CATALOG NO': this.releaseInfo.catalog_number,
          'ARTIST': this.releaseInfo.artist,
          'TITLE': this.releaseInfo.title,
          'GENRE': Array.isArray(this.releaseInfo.genre) ? this.releaseInfo.genre.join(', ') : this.releaseInfo.genre,
          'FORMAT': Array.isArray(this.releaseInfo.format) ? this.releaseInfo.format.join(', ') : this.releaseInfo.format,
          'LABEL': Array.isArray(this.releaseInfo.label) ? this.releaseInfo.label.join(', ') : this.releaseInfo.label,
          'RELEASED': this.releaseInfo.released_year,
          'PRICE_ORIGINAL': parseFloat(this.currencyPrice),
          'CURRENCY': this.selectedCurrency,
          'PRICE_TOKO': this.hargaTokoOtomatis,
          'PRICE_DISCOGS': this.convertedPriceIDR,
          'COVER_QUALITY': this.coverQuality,
          'CALCULATED_PRICE': this.calculatedPrice,
          'FINAL_PRICE': this.finalPrice,
        };

        return new Promise((resolve, reject) => {
          const transaction = this.db.transaction(this.objectStoreName, 'readwrite');
          const objectStore = transaction.objectStore(this.objectStoreName);
          const addRequest = objectStore.add({ data: recordToSave });

          addRequest.onsuccess = async () => {
            this.discogsCode = '';
            this.releaseInfo = null;
            this.selectedCurrency = 'USD';
            this.currencyPrice = null;
            this.convertedPriceIDR = null;
            this.coverQuality = null;
            this.calculatedPrice = null;
            this.finalPrice = null;
            await this.loadStoredData();
            this.newRecordCount++;
            this.showNotification('success', 'Data berhasil disimpan!');
            this.activeTab = 'view';
            resolve();
          };

          addRequest.onerror = (event) => {
            console.error('Gagal menyimpan ke IndexedDB:', event.target.error);
            this.showNotification('error', 'Gagal menyimpan data.');
            reject(event.target.error);
          };
        });
      } else {
        this.showNotification('warning', 'Harap isi harga dan kualitas cover.');
      }
    },
    formatCurrency(value) {
      if (value !== null) {
        return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0, maximumFractionDigits: 0 }).format(value);
      }
      return '';
    },
    async deleteRecord(id) {
      return new Promise((resolve, reject) => {
        const transaction = this.db.transaction(this.objectStoreName, 'readwrite');
        const objectStore = transaction.objectStore(this.objectStoreName);
        const deleteRequest = objectStore.delete(id);

        deleteRequest.onsuccess = async () => {
          await this.loadStoredData();
          this.showNotification('success', 'Data berhasil dihapus!');
          resolve();
        };

        deleteRequest.onerror = (event) => {
          console.error('Gagal menghapus data:', event.target.error);
          this.showNotification('error', 'Gagal menghapus data.');
          reject(event.target.error);
        };
      });
    },
    confirmDelete(id) {
      if (confirm('Apakah Anda yakin ingin menghapus data ini?')) {
        this.deleteRecord(id);
      }
    },
    async exportToCsv() {
      if (this.storedData.length === 0) {
        this.showNotification('warning', 'Tidak ada data untuk diekspor.');
        return;
      }

      const headerRows = [
        'NO (D00001),CATALOG NO,ARTIST,TITLE,GENRE,FORMAT,LABEL,RELEASED,HARGA ASLI,MATA UANG,HARGA TOKO,HARGA DISCOGS,KUALITAS COVER,HASIL KALKULASI,HARGA FINAL',
      ];
      let csvContent = headerRows.join('\n') + '\n';

      this.storedData.forEach((item, index) => {
        const data = item.data;
        const row = [
          `D${String(index + 1).padStart(5, '0')}`,
          data['CATALOG NO'] || '',
          `"${data.ARTIST || ''}"`,
          `"${data.TITLE || ''}"`,
          data.GENRE || '',
          data.FORMAT || '',
          data.LABEL || '',
          data.RELEASED || '',
          data.PRICE_ORIGINAL || '',
          data.CURRENCY || '',
          data.PRICE_TOKO || '',
          data.PRICE_DISCOGS || '',
          data.COVER_QUALITY || '',
          data.CALCULATED_PRICE || '',
          data.FINAL_PRICE || '',
        ];
        csvContent += row.join(',') + '\n';
      });

      const filename = 'discogs_records.csv';
      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', filename);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
      this.showNotification('success', 'Data berhasil diekspor ke CSV!');
    },
    async clearIndexedDB() {
      if (confirm('Apakah Anda yakin ingin menghapus semua data?')) {
        return new Promise((resolve, reject) => {
          const request = indexedDB.deleteDatabase('discogsDB');

          request.onsuccess = () => {
            console.log('Database berhasil dihapus.');
            this.db = null;
            this.storedData = [];
            this.activeTab = 'input';
            this.initDatabase();
            this.showNotification('success', 'Semua data berhasil dihapus!');
            resolve();
          };

          request.onerror = (event) => {
            console.error('Gagal menghapus database:', event.target.error);
            this.showNotification('error', 'Gagal menghapus database.');
            reject(event.target.error);
          };
        });
      }
    },
    showNotification(type, message) {
      this.notification.type = type;
      this.notification.message = message;
      this.notification.show = true;
      setTimeout(() => {
        this.notification.show = false;
      }, 3000);
    },
    clearNotification() {
      this.notification.show = false;
      this.notification.message = '';
      this.notification.type = '';
    },
    formatCurrency(value) {
      if (value !== null) {
        return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0, maximumFractionDigits: 0 }).format(value);
      }
      return '';
    },
  },
};
</script>

<style>
.tabs {
  margin-bottom: 20px;
}

.tabs button {
  padding: 10px 20px;
  margin-right: 10px;
  cursor: pointer;
  border: 1px solid #ccc;
  border-radius: 5px 5px 0 0;
  background-color: #f0f0f0;
  color: #333; /* Warna teks default */
}

.tabs button.active {
  background-color: #3b82f6; /* Warna biru Tailwind 500 (sesuaikan jika perlu) */
  color: white;
  border-color: #3b82f6; /* Warna border aktif */
}

.text-black {
  color: black;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

table th, table td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: left;
}

table th {
  background-color: #f0f0f0;
}

.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  padding: 15px 20px;
  border-radius: 5px;
  font-weight: bold;
  z-index: 10;
}

.notification.success {
  background-color: #4CAF50;
  color: white;
}

.notification.error {
  background-color: #F44336;
  color: white;
}

.notification.warning {
  background-color: #FF9800;
  color: white;
}

.tabs button .badge {
  background-color: green;
  color: white;
  border-radius: 50%;
  padding: 2px 5px;
  font-size: 0.8em;
  margin-left: 5px;
  vertical-align: middle;
}
</style>