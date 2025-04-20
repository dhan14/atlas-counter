<template>
    <div v-if="showInfo" class="bg-white rounded-lg shadow-md p-6 mt-4">
      <h2 class="text-lg font-semibold mb-4">Informasi Album</h2>
      <div class="mb-2">
        <strong class="text-gray-700">Artis:</strong>
        <span class="text-blue-500">{{ albumData.artis }}</span>
      </div>
      <div>
        <strong class="text-gray-700">Nama Album:</strong>
        <span class="text-green-500">{{ albumData.nama_album }}</span>
      </div>
    </div>
    <div v-else class="mt-4 text-gray-500 italic">
      Informasi album akan ditampilkan di sini setelah ID dimasukkan dan data ditemukan.
    </div>
  </template>
  
  <script setup>
  import { ref, reactive } from 'vue';
  
  // Properti (props) yang akan diterima dari komponen parent
  const props = defineProps({
    albumData: {
      type: Object,
      default: () => ({
        artis: '',
        nama_album: '',
      }),
    },
    showInfo: {
      type: Boolean,
      default: false,
    },
  });
  
  // Anda juga bisa menggunakan reactive jika data album kompleks
  // const albumData = reactive({
  //   artis: props.albumData.artis,
  //   nama_album: props.albumData.nama_album,
  // });
  
  // const showInfo = ref(props.showInfo);
//   </script>
  <style scoped>

  </style>