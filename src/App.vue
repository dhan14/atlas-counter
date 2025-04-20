<script setup>
import DiscogsInput from './components/DiscogsInput.vue';
// import AlbumInfo from './components/AlbumInfo.vue';
// import ScratchCode from './components/ScratchCode.vue';
</script>

<template>
  <div class="bg-gray-100 min-h-screen">
    <header class="bg-blue-500 text-white py-4">
      <div class="container mx-auto px-4">
        <h1 class="text-xl font-bold">Atlas Record</h1>
      </div>
    </header>

    <main class="container mx-auto py-8 px-4">
      <DiscogsInput />
      <!-- <AlbumInfo /> -->
      <!-- <ScratchCode /> -->
    </main>

    <footer class="bg-gray-800 text-gray-300 py-4 text-center">
      <p>&copy; 2025 Atlas Record</p>
    </footer>
  </div>
</template>

<!-- <script setup>
// Anda bisa menambahkan logika JavaScript di sini jika diperlukan
</script> -->

<!-- <style scoped>
/* Gaya khusus untuk komponen ini jika diperlukan */
</style> -->

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
