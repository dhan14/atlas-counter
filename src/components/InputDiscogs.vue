<template>
    <div class="bg-white rounded-lg shadow-md p-6 mb-4">
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
      <h2 class="text-lg font-semibold mb-4 text-gray-400">Masukkan ID Album Discogs</h2>
      <div class="mb-4">
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
        <div class="flex mx-8">
          <div class="w-1/3">
            <p class="text-black font-semibold text-left">CATALOG NO</p>
            <p class="text-black font-semibold text-left">ARTIST</p>
            <p class="text-black font-semibold text-left">TITLE</p>
            <p class="text-black font-semibold text-left">GENRE</p>
            <p class="text-black font-semibold text-left">FORMAT</p>
            <p class="text-black font-semibold text-left">LABEL</p>
            <p class="text-black font-semibold text-left">RELEASED</p>
            <p class="text-black font-semibold text-left">KONDISI COVER</p>
          </div>
          <div class="w-1/3 pr-3">
            <p class="text-black font-semibold text-right">:</p>
            <p class="text-black font-semibold text-right">:</p>
            <p class="text-black font-semibold text-right">:</p>
            <p class="text-black font-semibold text-right">:</p>
            <p class="text-black font-semibold text-right">:</p>
            <p class="text-black font-semibold text-right">:</p>
            <p class="text-black font-semibold text-right">:</p>
            <p class="text-black font-semibold text-right">:</p>
          </div>
          <div class="w-1/3">
            <p class="text-black text-left">{{ releaseInfo.catalog_number }}</p>
            <p class="text-black text-left">{{ releaseInfo.artist }}</p>
            <p class="text-black text-left">{{ releaseInfo.title }}</p>
            <p class="text-black text-left">{{ releaseInfo.genre ? releaseInfo.genre.join(', ') : '' }}</p>
            <p class="text-black text-left">{{ releaseInfo.format ? releaseInfo.format.join(', ') : '' }}</p>
            <p class="text-black text-left">{{ releaseInfo.label && releaseInfo.label.length > 0 ? releaseInfo.label[0] : '' }}</p>
            <p class="text-black text-left">{{ releaseInfo.released_year }}</p>
            <div class="flex items-center mb-2">
              <input type="radio" id="coverConditionCover" v-model="coverCondition" value="Cover" class="mr-2">
              <label class="text-gray-700" for="coverConditionCover">Cover</label>
            </div>
            <div class="flex items-center">
              <input type="radio" id="coverConditionNonCover" v-model="coverCondition" value="Non Cover" class="mr-2">
              <label class="text-gray-700" for="coverConditionNonCover">Non Cover</label>
            </div>
          </div>
        </div>
  
        <div class="mt-7 grid grid-cols-1 md:grid-cols-2 gap-4">
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
  
        <button @click="saveToIndexedDBWithCalculation"
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
      <div class="flex items-center mb-4">
        <button @click="clearIndexedDB" :disabled="storedData.length === 0"
          class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline mr-2">
          Hapus Semua Data
        </button>
        <button @click="toggleEditMode"
          class="bg-yellow-500 hover:bg-yellow-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
          {{ isEditMode ? 'Selesai Edit' : 'Mode Edit' }}
        </button>
        <button v-if="isEditMode && selectedToDelete.length > 0" @click="deleteSelectedRecords"
          class="bg-red-600 hover:bg-red-800 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline ml-2">
          Hapus Terpilih ({{ selectedToDelete.length }})
        </button>
      </div>
      <div class="overflow-x-auto">
        <table v-if="storedData.length > 0" class="w-full border-collapse">
            <thead>
  <tr class="text-gray-700">
    <th v-if="isEditMode" class="border border-blue-300 p-2 text-left">
      <input type="checkbox" @change="selectAllToDelete">
    </th>
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
    <th v-if="isEditMode" class="border border-gray-300 p-2 text-left" rowspan="2">Aksi</th>
  </tr>
  <tr class="text-gray-700">
    <th class="border border-gray-300 p-2 text-left">Media</th>
    <th class="border border-gray-300 p-2 text-left">Cover</th>
  </tr>
</thead>
<tbody>
  <tr v-for="(item, index) in storedData" :key="item.id">
    <td v-if="isEditMode" class="border border-gray-300 p-2 text-black">
      <input type="checkbox" :value="item.id" @change="toggleDelete(item.id)">
    </td>
    <td class="border border-gray-300 p-2 text-black">{{ index + 1 }}</td>
    <td class="border border-gray-300 p-2 text-black" @click="startEdit(item.id, 'CATALOG NO')">
      <span v-if="!isEditMode || editingRecordId !== item.id">{{ item.data['CATALOG NO'] }}</span>
      <input v-else type="text" class="edit-input" v-model="editedValues['CATALOG NO']">
    </td>
    <td class="border border-gray-300 p-2 text-black" @click="startEdit(item.id, 'ARTIST')">
      <span v-if="!isEditMode || editingRecordId !== item.id">{{ item.data.ARTIST }}</span>
      <input v-else type="text" class="edit-input" v-model="editedValues.ARTIST">
    </td>
    <td class="border border-gray-300 p-2 text-black" @click="startEdit(item.id, 'TITLE')">
      <span v-if="!isEditMode || editingRecordId !== item.id">{{ item.data.TITLE }}</span>
      <input v-else type="text" class="edit-input" v-model="editedValues.TITLE">
    </td>
    <td class="border border-gray-300 p-2 text-black" @click="startEdit(item.id, 'GENRE')">
      <span v-if="!isEditMode || editingRecordId !== item.id">{{ item.data.GENRE }}</span>
      <input v-else type="text" class="edit-input" v-model="editedValues.GENRE">
    </td>
    <td class="border border-gray-300 p-2 text-black" @click="startEdit(item.id, 'FORMAT')">
      <span v-if="!isEditMode || editingRecordId !== item.id">{{ item.data.FORMAT }}</span>
      <input v-else type="text" class="edit-input" v-model="editedValues.FORMAT">
    </td>
    <td class="border border-gray-300 p-2 text-black" @click="startEdit(item.id, 'LABEL')">
      <span v-if="!isEditMode || editingRecordId !== item.id">{{ item.data.LABEL && item.data.LABEL[0] }}</span>
      <input v-else type="text" class="edit-input" v-model="editedValues.LABEL">
    </td>
    <td class="border border-gray-300 p-2 text-black" @click="startEdit(item.id, 'RELEASED')">
      <span v-if="!isEditMode || editingRecordId !== item.id">{{ item.data.RELEASEED }}</span>
      <input v-else type="text" class="edit-input" v-model="editedValues.RELEASEED">
    </td>
    <td class="border border-gray-300 p-2 text-black"></td>
    <td class="border border-gray-300 p-2 text-black">{{ item.data.COVER_CONDITION }}</td>
    <td class="border border-gray-300 p-2 text-black" @click="startEdit(item.id, 'HARGA_AKHIR')">
      <span v-if="!isEditMode || editingRecordId !== item.id">{{ formatCurrency(item.data.HARGA_AKHIR) }}</span>
      <input v-else type="number" class="edit-input" v-model.number="editedValues.HARGA_AKHIR">
    </td>
    <td v-if="isEditMode" class="border border-gray-300 p-2">
      <template v-if="editingRecordId === item.id">
        <div class="edit-actions">
          <button @click="saveEdit(item.id)" class="save">Simpan</button>
          <button @click="cancelEdit()" class="cancel">Batal</button>
        </div>
      </template>
      <button v-else @click="confirmDelete(item.id)"
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

  </div>
</template>

<script>
export default {
  data() {
    return {
      coverCondition: 'Cover',
      hargaMode: 'otomatis',
      hargaManual: null,
      kualitasCoverInput: null,
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
      isEditMode: false,
      selectedToDelete: [],
      selectAll: false,
      editingRecordId: null,
      editedValues: {},
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
        this.selectedCurrency = 'USD';
        this.currencyPrice = null;
        this.convertedPriceIDR = null;
        this.kualitasCoverInput = null;
        this.coverQuality = null;
      }
      this.calculateFinalPrice();
    },
    currencyPrice() {
      this.convertCurrency();
      this.calculateFinalPrice();
    },
    selectedCurrency() {
      this.currencyPrice = null;
      this.convertedPriceIDR = null;
      this.calculateFinalPrice();
    },
    kualitasCoverInput(newVal) {
      if (this.hargaMode === 'otomatis' && newVal !== null) {
        this.coverQuality = newVal / 100;
        this.calculateFinalPrice();
      } else {
        this.coverQuality = null;
        this.finalPrice = null;
      }
    },
    hargaManual(newVal) {
      if (this.hargaMode === 'manual' && newVal !== null) {
        this.finalPrice = newVal;
      } else {
        this.calculateFinalPrice();
      }
    },
    releaseInfo() {
      this.calculateFinalPrice();
    },
    convertedPriceIDR() {
      this.calculateFinalPrice();
    },
  },
  computed: {
    hargaTokoOtomatis() {
      if (this.releaseInfo && typeof this.releaseInfo.format === 'string') {
        const format = this.releaseInfo.format.trim();
        if (format.startsWith('LP')) {
          return 150000;
        } else if (format.startsWith('12"')) {
          return 100000;
        }
      }
      return 0;
    },
  },
  methods: {
    setActiveTab(tabName) {
      this.activeTab = tabName;
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
              LABEL: item.data.LABEL
                ? Array.isArray(item.data.LABEL) && item.data.LABEL.length > 0
                  ? [item.data.LABEL[0]]
                  : [item.data.LABEL]
                : [],
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
        const response = await fetch(`http://127.0.0.1:5000/discogs/release_info/${this.discogsCode}`);
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
      this.dollarToRupiahRate = 16000;
      this.euroToRupiahRate = 19000;
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
    async saveToIndexedDBWithCalculation() {
      let priceToSave = null;
      let currencyToSave = 'IDR';

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
        'NO;CATALOG NO;ARTIST;TITLE;GENRE;FORMAT;LABEL;RELEASED;MEDIA;COVER;PRICE',
      ];
      let csvContent = headerRows.join('\n') + '\n';

      this.storedData.forEach((item, index) => {
        const data = item.data;
        const row = [
          `${index + 1}`,
          `"${Array.isArray(data['CATALOG NO']) ? data['CATALOG NO'].join(',') : data['CATALOG NO'] || ''}"`,
          `"${data.ARTIST || ''}"`,
          `"${data.TITLE || ''}"`,
          `"${Array.isArray(data.GENRE) ? data.GENRE.join(',') : data.GENRE || ''}"`,
          `"${Array.isArray(data.FORMAT) ? data.FORMAT.join(',') : data.FORMAT || ''}"`,
          `"${Array.isArray(data.LABEL) && data.LABEL.length > 0 ? data.LABEL[0] : data.LABEL || ''}"`,
          `"${data.RELEASEED || ''}"`,
          `"${data.MEDIA || ''}"`,
          `"${data.COVER_CONDITION || ''}"`,
          (data.HARGA_AKHIR ? `Rp${Number(data.HARGA_AKHIR).toLocaleString('id-ID')}` : ''),
        ];
        csvContent += row.join(';') + '\n';
      });

      const today = new Date();
      const year = today.getFullYear();
      const month = String(today.getMonth() + 1).padStart(2, '0');
      const day = String(today.getDate()).padStart(2, '0');
      const dateString = `<span class="math-inline">\{year\}\-</span>{month}-${day}`;
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
      this.showNotification('success', 'Data berhasil diekspor ke CSV (pemisah titik koma)!');
    },
    async clearIndexedDB() {
      if (confirm('Apakah Anda yakin ingin menghapus semua data?')) {
        return new Promise((resolve, reject) => {
          const dbNameToDelete = 'discogsDB';
          if (this.db) {
            this.db.close();
            this.db = null;
          }
          const request = indexedDB.deleteDatabase(dbNameToDelete);
          request.onsuccess = () => {
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
    toggleEditMode() {
      this.isEditMode = !this.isEditMode;
      this.selectedToDelete = [];
      this.selectAll = false;
      this.editingRecordId = null;
      this.editedValues = {};
    },
    toggleDelete(id) {
      const index = this.selectedToDelete.indexOf(id);
      if (index > -1) {
        this.selectedToDelete.splice(index, 1);
      } else {
        this.selectedToDelete.push(id);
      }
    },
    selectAllToDelete() {
      this.selectAll = !this.selectAll;
      this.selectedToDelete = this.selectAll ? this.storedData.map(item => item.id) : [];
    },
    async deleteSelectedRecords() {
      if (this.selectedToDelete.length > 0) {
        if (confirm(`Apakah Anda yakin ingin menghapus ${this.selectedToDelete.length} data yang dipilih?`)) {
          for (const id of this.selectedToDelete) {
            await this.deleteRecord(id);
          }
          this.selectedToDelete = [];
          this.selectAll = false;
        }
      } else {
        this.showNotification('warning', 'Tidak ada data yang dipilih untuk dihapus.');
      }
    },
    startEdit(itemId, field) {
      this.editingRecordId = itemId;
      const record = this.storedData.find(item => item.id === itemId);
      if (record && record.data) {
        this.editedValues = { ...record.data };
      }
    },
    async saveEdit(itemId) {
  console.log('saveEdit dipanggil untuk itemId:', itemId);
  console.log('editedValues saat menyimpan:', this.editedValues);
  if (!this.editedValues) return;
  const transaction = this.db.transaction(this.objectStoreName, 'readwrite');
  const objectStore = transaction.objectStore(this.objectStoreName);
  const getRequest = objectStore.get(itemId);

  getRequest.onsuccess = async (event) => {
    const record = event.target.result;
    if (record) {
      console.log('Record sebelum diupdate:', record);
      record.data = { ...this.editedValues };
      console.log('Record setelah diupdate:', record);
      const updateRequest = objectStore.put(record);
      updateRequest.onsuccess = async () => {
        console.log('Data berhasil diupdate di IndexedDB');
        this.editingRecordId = null;
        this.editedValues = {};
        await this.loadStoredData();
        this.showNotification('success', 'Data berhasil diperbarui!');
      };
      updateRequest.onerror = (event) => {
        console.error('Gagal memperbarui data:', event.target.error);
        this.showNotification('error', 'Gagal memperbarui data.');
      };
    }
  };

  getRequest.onerror = (event) => {
    console.error('Gagal mengambil data untuk diedit:', event.target.error);
    this.showNotification('error', 'Gagal mengambil data untuk diedit.');
  };
}
  },
};
</script>

<style>
/* ... (style yang sudah ada) */
.edit-input {
  width: 100%;
  padding: 6px;
  margin-bottom: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}
.edit-actions button {
  margin-right: 5px;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
}
.edit-actions .save {
  background-color: #4CAF50;
  color: white;
  border: none;
}
.edit-actions .cancel {
  background-color: #f44336;
  color: white;
  border: none;
}
</style>