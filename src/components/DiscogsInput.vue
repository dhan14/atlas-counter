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
          <div v-if="notification.show && notification.type === 'success'" class="notification success">{{
            notification.message }}</div>
          <div v-if="notification.show && notification.type === 'error'" class="notification error">{{
            notification.message }}</div>
          <h2 class="text-lg font-semibold mb-4">Masukkan ID Album Discogs</h2>
          <div class="mb-4">
            <label for="discogsCode" class="block text-gray-700 text-sm font-bold mb-2">Kode Discogs:</label>
            <label for="discogsCode" class="block text-gray-700 text-sm font-bold mb-2">
              Contoh, Salin kode yang di Highlight ke form di bawah:
              <a href="https://www.discogs.com/release/" class="text-gray-400">https://www.discogs.com/release/</a>
              <span class="font-bold text-black bg-green-300">221824</span>
              <span class="text-gray-400">-Rick-Astley-Never-Gonna-Give-You-Up</span>
            </label>
            <input type="text" id="discogsCode" v-model="discogsCode"
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
              placeholder="Contoh: 221824" />
          </div>
          <button @click="fetchReleaseInfo"
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
            Cari
          </button>
  
          <div v-if="releaseInfo" class="mt-4">
            <h3>Preview Data</h3>
            <p class="text-black"><strong>CATALOG NO:</strong> {{ releaseInfo.catalog_number }}</p>
            <p class="text-black"><strong>ARTIST:</strong> {{ releaseInfo.artist }}</p>
            <p class="text-black"><strong>TITLE:</strong> {{ releaseInfo.title }}</p>
            <p class="text-black"><strong>GENRE:</strong> {{ releaseInfo.genre ? releaseInfo.genre.join(', ') : '' }}</p>
            <p class="text-black"><strong>FORMAT:</strong> {{ releaseInfo.format ? releaseInfo.format.join(', ') : '' }}
            </p>
            <p class="text-black"><strong>LABEL:</strong> {{ releaseInfo.label ? releaseInfo.label.join(', ') : '' }}</p>
            <p class="text-black"><strong>RELEASED:</strong> {{ releaseInfo.released_year }}</p>
            <div class="mt-4">
        <h3 class="text-gray-700">Kondisi Cover</h3>
        <div class="flex items-center mb-2">
          <input type="radio" id="coverConditionCover" v-model="coverCondition" value="Cover" class="mr-2">
          <label class="text-gray-700" for="coverConditionCover">Cover</label>
        </div>
        <div class="flex items-center">
          <input type="radio" id="coverConditionNonCover" v-model="coverCondition" value="Non Cover" class="mr-2">
          <label class="text-gray-700" for="coverConditionNonCover">Non Cover</label>
        </div>
      </div>
            
  
            <div class="mt-4 grid grid-cols-1 md:grid-cols-2 gap-4">
              <!-- INPUT HARGA OTOMATIS -->
              <div class="border rounded p-4" :class="{ 'bg-gray-400': hargaMode === 'otomatis' }">
                <label class="block font-bold mb-2 text-gray-700">
                  <input type="radio" v-model="hargaMode" value="otomatis" class="mr-2"> Harga Otomatis
                </label>
                <div v-if="hargaMode === 'otomatis' || hargaMode === 'manual'" class="mt-2">
                  <label :for="selectedCurrency === 'USD' ? 'priceUSD' : 'priceEUR'"
                        class="block text-gray-700 text-sm font-semibold mb-1">Harga {{ selectedCurrency === 'USD' ?
                        'Dollar' : 'Euro' }} Di Discogs:</label>
                  <div class="flex items-center gap-1">
                    <div>
                      <select id="currency" v-model="selectedCurrency"
                        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm"
                        :disabled="hargaMode === 'manual'" style="width: auto;">
                        <option value="USD">(USD)</option>
                        <option value="EUR">(EUR)</option>
                      </select>
                    </div>
  
                    <div class="flex-1">
                      <div class="flex items-center">
                        <input type="number" :id="selectedCurrency === 'USD' ? 'priceUSD' : 'priceEUR'"
                          v-model="currencyPrice" step="0.01"
                          class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm"
                          :disabled="hargaMode === 'manual'" />
                        <span
                          class="inline-flex items-center px-3 rounded border border-l-0 bg-gray-50 text-gray-500 text-sm p-2">
                          {{ selectedCurrency === 'USD' ? '$' : '€' }}
                        </span>
                      </div>
                    </div>
                  </div>
  
                  <label for="priceIDR" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Setelah Konversi ke
                    IDR:</label>
                    <input type="text" id="priceIDR" :value="formatCurrency(convertedPriceIDR)" readonly
                    class="appearance-none w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm text-center"
                    :disabled="hargaMode === 'manual'" />
  
                  <label for="kualitasCoverInput" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Kualitas
                    Cover (1-100):</label>
                  <div class="flex">
                    <input type="number" id="kualitasCoverInput" v-model.number="kualitasCoverInput" min="1" max="100"
                      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm"
                      :disabled="hargaMode === 'manual'" />
                    <span
                      class="inline-flex items-center px-3 rounded border border-l-0 bg-gray-50 text-gray-500 text-sm">
                      %
                    </span>
                  </div>
  
                  <label for="finalPriceOtomatis" class="block text-gray-700 text-sm font-semibold mt-2 mb-1">Harga
                    Akhir:</label>
                  <input type="text" id="finalPriceOtomatis" :value="formatCurrency(finalPrice)" readonly
                    class="appearance-none w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm text-center"
                    :disabled="hargaMode === 'manual'" />
                </div>
              </div>
  
              <!-- INPUT HARGA MANUAL -->
  
              <div class="border rounded p-4" :class="{ 'bg-gray-400': hargaMode === 'manual' }">
                <label class="block font-bold mb-2 text-gray-700">
                  <input type="radio" v-model="hargaMode" value="manual" class="mr-2 text-gray-700"> Add Harga Manual
                </label>
                <div v-if="hargaMode === 'manual' || hargaMode === 'otomatis'">
                  <label for="hargaManual" class="block text-sm font-semibold mt-2 mb-1">Masukkan Harga Manual
                    (IDR):</label>
                  <div class="flex">
                    <span
                      class="inline-flex items-center px-3 rounded border border-l-0 bg-gray-50 text-gray-500 text-sm">
                      Rp
                    </span>
                    <input type="number" id="hargaManual" v-model.number="hargaManual"
                      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline text-sm"
                      :disabled="hargaMode === 'otomatis'" />
                  </div>
                </div>
              </div>
            </div>
  
            <button @click="saveToIndexedDBWithCalculation" :disabled="!currencyPrice || !coverQuality"
              class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline mt-4">
              Simpan
            </button>
          </div>
          <p v-else-if="fetchError" class="mt-4 text-red-500">{{ fetchError }}</p>
          <p v-else class="mt-4 text-gray-500 italic">Masukkan kode Discogs dan klik "Cari".</p>
        </div>
        
        <div v-if="activeTab === 'view'" class="bg-gray rounded-lg shadow-md p-6 mt-4">
          <div v-if="notification.show && notification.type === 'success'" class="notification success">{{
            notification.message }}</div>
          <div v-if="notification.show && notification.type === 'error'" class="notification error">{{
            notification.message }}</div>

          <h2 class="text-lg font-semibold mb-4 text-gray-700">Data Tersimpan</h2>
          <div class="overflow-x-auto">
            <table v-if="storedData.length > 0" class="w-full border-collapse">
              <thead>
      <tr class="text-gray-700">
        <th class="border border-blue-300 p-2 text-left" rowspan="2">No</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Catalog No</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Artist</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Title</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Genre</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Format</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Label</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Released</th>
        <th class="border border-gray-300 p-2 text-left" colspan="2">Condition</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Price</th>
        <th class="border border-gray-300 p-2 text-left" rowspan="2">Aksi</th>
      </tr>
      <tr class="text-gray-700">
        <th class="border border-gray-300 p-2 text-left">Media</th>
        <th class="border border-gray-300 p-2 text-left">Cover</th>
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
        <td class="border border-gray-300 p-2 text-black"></td>
        <td class="border border-gray-300 p-2 text-black">{{ item.data.COVER_CONDITION }}</td>
        <td class="border border-gray-300 p-2 text-black">{{ formatCurrency(item.data.HARGA_AKHIR) }}</td>
        <td class="border border-gray-300 p-2">
          <button @click="confirmDelete(item.id)"
            class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-2 rounded focus:outline-none focus:shadow-outline">Hapus</button>
        </td>
      </tr>
    </tbody>
            </table>
          <p v-else class="text-gray-500 italic">Tidak ada data tersimpan.</p>
          </div>
          
          <button @click="exportToCsv" :disabled="storedData.length === 0"
            class="bg-indigo-500 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline mt-4 mr-2">
            Ekspor ke CSV
          </button>
          <button @click="clearIndexedDB" :disabled="storedData.length === 0"
            class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline mt-4">
            Kosongkan Data
          </button>
        </div>
    </div>
  </template>


<script>
export default {
  data() {
    return {
      coverCondition: 'Cover', // Default value
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
    hargaMode(newVal) {
      if (newVal === 'otomatis') {
        this.hargaManual = null;
      } else if (newVal === 'manual') {
        this.selectedCurrency = 'USD'; // Reset mata uang
        this.currencyPrice = null;
        this.convertedPriceIDR = null;
        this.kualitasCoverInput = null;
        this.coverQuality = null;
      }
      this.calculateFinalPrice(); // Panggil saat mode berubah
    },
    currencyPrice() {
      this.convertCurrency();
      this.calculateFinalPrice(); // Panggil saat harga berubah
    },
    selectedCurrency() {
      this.currencyPrice = null;
      this.convertedPriceIDR = null;
      this.calculateFinalPrice(); // Panggil saat mata uang berubah
    },
    kualitasCoverInput(newVal) {
      if (this.hargaMode === 'otomatis' && newVal !== null) {
        this.coverQuality = newVal / 100; // Konversi ke 0.01 - 1
        this.calculateFinalPrice(); // Panggil saat kualitas cover berubah
      } else {
        this.coverQuality = null;
        this.finalPrice = null;
      }
    },
    hargaManual(newVal) {
      if (this.hargaMode === 'manual' && newVal !== null) {
        this.finalPrice = newVal;
      } else {
        this.calculateFinalPrice(); // Panggil saat harga manual berubah
      }
    },
    releaseInfo() {
      this.calculateFinalPrice(); // Panggil saat info rilis berubah
    },
    convertedPriceIDR() {
      this.calculateFinalPrice(); // Panggil saat harga IDR hasil konversi berubah
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
    // calculatePrices() {
    //   if (this.convertedPriceIDR !== null && this.coverQuality !== null) {
    //     const hargaToko = this.hargaTokoOtomatis;
    //     const hargaDiscogs = this.convertedPriceIDR;

    //     let hargaTengah = hargaToko;
    //     if (hargaDiscogs > hargaToko) {
    //       hargaTengah = hargaToko + (hargaDiscogs - hargaToko) / 2;
    //     }

    //     this.calculatedPrice = hargaTengah * this.coverQuality;
    //     this.finalPrice = Math.round(this.calculatedPrice / 5000) * 5000;
    //   } else {
    //     this.calculatedPrice = null;
    //     this.finalPrice = null;
    //   }
    // },
    async saveToIndexedDBWithCalculation() {
      if (this.releaseInfo && this.convertedPriceIDR !== null && this.coverQuality !== null) {
        const recordToSave = {
  'CATALOG NO': this.releaseInfo ? this.releaseInfo.catalog_number : '',
  'ARTIST': this.releaseInfo ? this.releaseInfo.artist : '',
  'TITLE': this.releaseInfo ? this.releaseInfo.title : '',
  'GENRE': this.releaseInfo && Array.isArray(this.releaseInfo.genre) ? this.releaseInfo.genre.join(', ') : '',
  'FORMAT': this.releaseInfo && Array.isArray(this.releaseInfo.format) ? this.releaseInfo.format.join(', ') : '',
  'LABEL': this.releaseInfo && Array.isArray(this.releaseInfo.label) ? this.releaseInfo.label.join(', ') : '',
  'RELEASED': this.releaseInfo ? this.releaseInfo.released_year : '',
  'HARGA_MODE': this.hargaMode,
  'HARGA_ASLI': this.hargaMode === 'otomatis' ? parseFloat(this.currencyPrice) : null,
  'MATA_UANG': this.hargaMode === 'otomatis' ? this.selectedCurrency : null,
  'HARGA_MANUAL': this.hargaMode === 'manual' ? parseFloat(this.hargaManual) : null,
  'HARGA_TOKO': this.hargaTokoOtomatis,
  'HARGA_DISCOGS': this.hargaMode === 'otomatis' ? this.convertedPriceIDR : null,
  'KUALITAS_COVER': this.coverQuality,
  'HARGA_AKHIR': priceToSave,
  'COVER_CONDITION': this.coverCondition,
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
  'NO,CATALOG NO,ARTIST,TITLE,GENRE,FORMAT,LABEL,RELEASED,MEDIA,COVER,PRICE',
    ];
      let csvContent = headerRows.join('\n') + '\n';

      this.storedData.forEach((item, index) => {
        const data = item.data;
        
        const row = [
        `D${String(index + 1).padStart(5, '0')}`,
        Array.isArray(data['CATALOG NO']) ? data['CATALOG NO'].join(';') : data['CATALOG NO'] || '',
        `"${data.ARTIST || ''}"`,
        `"${data.TITLE || ''}"`,  
        `"${Array.isArray(data.GENRE) ? data.GENRE.join(';') : data.GENRE || ''}"`, // Tambah kutip
        `"${Array.isArray(data.FORMAT) ? data.FORMAT.join(';') : data.FORMAT || ''}"`, // Tambah kutip
        data.LABEL || '',
        data.RELEASED || '',
        data.MEDIA || '',
        data.COVER_CONDITION || '', // Gunakan nama properti yang benar
        data.HARGA_AKHIR || '',    // Gunakan nama properti yang benar
];
        csvContent += row.join(',') + '\n';
      });
      
        // Dapatkan tanggal hari ini
      const today = new Date();
      const year = today.getFullYear();
      const month = String(today.getMonth() + 1).padStart(2, '0'); // Month dimulai dari 0
      const day = String(today.getDate()).padStart(2, '0');
      const dateString = `${year}-${month}-${day}`;

      const filename = `discogs_records_${dateString}.csv`;
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
    calculateFinalPrice() {
    if (this.hargaMode === 'otomatis') {
        let basePrice = 0;
        if (this.releaseInfo && this.releaseInfo.format) {
          basePrice = this.releaseInfo.format.includes('LP') ? 150000 : 100000;
          if (this.convertedPriceIDR !== null && this.convertedPriceIDR > basePrice) {
            basePrice = this.convertedPriceIDR;
          }
        }

        if (basePrice > 0 && this.coverQuality !== null) {
          this.finalPrice = Math.round(basePrice * this.coverQuality / 5000) * 5000;
        } else {
          this.finalPrice = null;
        }
      } else if (this.hargaMode === 'manual' && this.hargaManual !== null) {
        this.finalPrice = this.hargaManual;
      } else {
        this.finalPrice = null;
      }
    },
    async saveToIndexedDBWithCalculation() {
  let priceToSave = null;
  let currencyToSave = 'IDR'; // Default jika manual

  if (this.hargaMode === 'otomatis' && this.finalPrice !== null && this.currencyPrice !== null && this.coverQuality !== null) {
    priceToSave = this.finalPrice;
    currencyToSave = this.selectedCurrency;
  } else if (this.hargaMode === 'manual' && this.finalPrice !== null && this.hargaManual !== null) {
    priceToSave = this.finalPrice;
  } else {
    this.showNotification('warning', 'Harap isi semua informasi harga yang diperlukan.');
    return;
  }

  const recordToSave = {
    'CATALOG NO': this.releaseInfo ? this.releaseInfo.catalog_number : '',
    'ARTIST': this.releaseInfo ? this.releaseInfo.artist : '',
    'TITLE': this.releaseInfo ? this.releaseInfo.title : '',
    'GENRE': this.releaseInfo && Array.isArray(this.releaseInfo.genre) ? this.releaseInfo.genre.join(', ') : '',
    'FORMAT': this.releaseInfo && Array.isArray(this.releaseInfo.format) ? this.releaseInfo.format.join(', ') : '',
    'LABEL': this.releaseInfo && Array.isArray(this.releaseInfo.label) ? this.releaseInfo.label.join(', ') : '',
    'RELEASED': this.releaseInfo ? this.releaseInfo.released_year : '',
    'HARGA_MODE': this.hargaMode,
    'HARGA_ASLI': this.hargaMode === 'otomatis' ? parseFloat(this.currencyPrice) : null,
    'MATA_UANG': this.hargaMode === 'otomatis' ? this.selectedCurrency : null,
    'HARGA_MANUAL': this.hargaMode === 'manual' ? parseFloat(this.hargaManual) : null,
    'HARGA_TOKO': this.hargaTokoOtomatis,
    'HARGA_DISCOGS': this.hargaMode === 'otomatis' ? this.convertedPriceIDR : null,
    'KUALITAS_COVER': this.coverQuality,
    'HARGA_AKHIR': priceToSave,
    'COVER_CONDITION': this.coverCondition,
  };

  return new Promise((resolve, reject) => {
    const transaction = this.db.transaction(this.objectStoreName, 'readwrite');
    const objectStore = transaction.objectStore(this.objectStoreName);
    const addRequest = objectStore.add({ data: recordToSave });

    addRequest.onsuccess = async () => {
      this.discogsCode = '';
      this.releaseInfo = null;
      this.hargaMode = 'otomatis';
      this.currencyPrice = null;
      this.selectedCurrency = 'USD';
      this.convertedPriceIDR = null;
      this.kualitasCoverInput = null;
      this.coverQuality = null;
      this.hargaManual = null;
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
  color: #333;
  /* Warna teks default */
}

.tabs button.active {
  background-color: #3b82f6;
  /* Warna biru Tailwind 500 (sesuaikan jika perlu) */
  color: white;
  border-color: #3b82f6;
  /* Warna border aktif */
}

.text-black {
  color: black;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

table th,
table td {
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

input[type="radio"] {
  /* Reset tampilan default browser */
  appearance: none;
  width: 16px;
  height: 16px;
  border: 2px solid #6b7280; /* Warna abu-abu Tailwind gray-700 */
  border-radius: 50%;
  background-color: white;
  outline: none;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

input[type="radio"]:checked {
  background-color: #6b7280; /* Warna abu-abu Tailwind gray-700 saat dipilih */
  border-color: #6b7280;
}

input[type="radio"]:checked::before {
  content: '';
  display: block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: white;
}
</style>
