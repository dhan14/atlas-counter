<template>
  <div>
    <h1>Aplikasi Input Data Discogs</h1>

    <div class="tabs">
      <button :class="{ active: activeTab === 'input' }" @click="activeTab = 'input'">Input Data</button>
      <button :class="{ active: activeTab === 'view' }" @click="activeTab = 'view'">Lihat Data</button>
    </div>

    <div v-if="activeTab === 'input'">
      <h2>Input Data</h2>
      <div>
        <label for="discogsCode">Kode Discogs:</label>
        <input type="text" id="discogsCode" v-model="discogsCode" />
        <button @click="fetchReleaseInfo">Cari</button>
      </div>

      <div v-if="releaseInfo">
        <h3>Preview Data</h3>
        <p><strong>CATALOG NO:</strong> {{ releaseInfo.catalog_number }}</p>
        <p><strong>ARTIST:</strong> {{ releaseInfo.artist }}</p>
        <p><strong>TITLE:</strong> {{ releaseInfo.title }}</p>
        <p><strong>GENRE:</strong> {{ releaseInfo.genre ? releaseInfo.genre.join(', ') : '' }}</p>
        <p><strong>FORMAT:</strong> {{ releaseInfo.format ? releaseInfo.format.join(', ') : '' }}</p>
        <p><strong>LABEL:</strong> {{ releaseInfo.label ? releaseInfo.label.join(', ') : '' }}</p>
        <p><strong>RELEASED:</strong> {{ releaseInfo.released_year }}</p>

        <div>
          <label for="price">Harga:</label>
          <input type="number" id="price" v-model="price" step="0.01" />
        </div>
        <button @click="saveToIndexedDB" :disabled="!price">Simpan</button>
      </div>
      <p v-else-if="fetchError">{{ fetchError }}</p>
      <p v-else>Masukkan kode Discogs dan klik "Cari".</p>
    </div>

    <div v-if="activeTab === 'view'">
      <h2>Lihat Data Tersimpan</h2>
      <table v-if="storedData.length > 0">
        <thead>
          <tr>
            <th>No</th>
            <th>Catalog No</th>
            <th>Artist</th>
            <th>Title</th>
            <th>Genre</th>
            <th>Format</th>
            <th>Label</th>
            <th>Released</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in storedData" :key="item.id">
            <td>{{ index + 1 }}</td>
            <td>{{ item.data['CATALOG NO'] }}</td>
            <td>{{ item.data.ARTIST }}</td>
            <td>{{ item.data.TITLE }}</td>
            <td>{{ item.data.GENRE ? item.data.GENRE.join(', ') : '' }}</td>
            <td>{{ item.data.FORMAT ? item.data.FORMAT.join(', ') : '' }}</td>
            <td>{{ item.data.LABEL ? item.data.LABEL.join(', ') : '' }}</td>
            <td>{{ item.data.RELEASED }}</td>
            <td>{{ item.data.PRICE }}</td>
          </tr>
        </tbody>
      </table>
      <p v-else>Tidak ada data tersimpan.</p>

      <button @click="exportToCsv" :disabled="storedData.length === 0">Ekspor ke CSV</button>
      <button @click="clearIndexedDB" :disabled="storedData.length === 0">Kosongkan Data</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeTab: 'input', // Tab aktif: 'input' atau 'view'
      db: null,
      objectStoreName: 'discogsReleases',
      discogsCode: '',
      releaseInfo: null,
      fetchError: null,
      price: null,
      storedData: [],
    };
  },
  async mounted() {
    await this.initDatabase();
    await this.loadStoredData();
  },
  methods: {
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
          this.storedData = getAllRequest.result;
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
      this.price = null; // Reset harga saat mencari data baru
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
      }
    },
    async saveToIndexedDB() {
      if (this.releaseInfo && this.price !== null) {
        const recordToSave = {
          'CATALOG NO': this.releaseInfo.catalog_number,
          'ARTIST': this.releaseInfo.artist,
          'TITLE': this.releaseInfo.title,
          'GENRE': this.releaseInfo.genre,
          'FORMAT': this.releaseInfo.format,
          'LABEL': this.releaseInfo.label,
          'RELEASED': this.releaseInfo.released_year,
          'PRICE': parseFloat(this.price), // Pastikan harga adalah number
        };

        return new Promise((resolve, reject) => {
          const transaction = this.db.transaction(this.objectStoreName, 'readwrite');
          const objectStore = transaction.objectStore(this.objectStoreName);
          const addRequest = objectStore.add({ data: recordToSave });

          addRequest.onsuccess = async () => {
            this.discogsCode = '';
            this.releaseInfo = null;
            this.price = null;
            await this.loadStoredData();
            this.activeTab = 'view'; // Pindah ke tab lihat data setelah menyimpan
            resolve();
          };

          addRequest.onerror = (event) => {
            console.error('Gagal menyimpan ke IndexedDB:', event.target.error);
            reject(event.target.error);
          };
        });
      } else if (!this.price) {
        alert('Harap masukkan harga sebelum menyimpan.');
      }
    },
    async exportToCsv() {
      if (this.storedData.length === 0) {
        alert('Tidak ada data untuk diekspor.');
        return;
      }

      const headerRows = [
        'NO (D00001),CATALOG NO,ARTIST,TITLE,GENRE,FORMAT,LABEL,RELEASED,CONDITION,,PRICE,,,,,',
        ',,,,,,,,MEDIA,COVER,,,,,,',
      ];
      let csvContent = headerRows.join('\n') + '\n';

      this.storedData.forEach((item, index) => {
        const data = item.data;
        const row = [
          `D${String(index + 1).padStart(5, '0')}`,
          data['CATALOG NO'] || '',
          `"${data.ARTIST || ''}"`,
          `"${data.TITLE || ''}"`,
          data.GENRE ? `"${data.GENRE.join('; ')}"` : '',
          data.FORMAT ? `"${data.FORMAT.join('; ')}"` : '',
          data.LABEL ? `"${data.LABEL.join('; ')}"` : '',
          data.RELEASED || '',
          '', // CONDITION MEDIA
          '', // CONDITION COVER
          data.PRICE || '',
          '', '', '', '', '', // Kolom kosong tambahan
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
            this.initDatabase(); // Inisialisasi database lagi
            resolve();
          };

          request.onerror = (event) => {
            console.error('Gagal menghapus database:', event.target.error);
            reject(event.target.error);
          };
        });
      }
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
}

.tabs button.active {
  background-color: #ddd;
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
</style>