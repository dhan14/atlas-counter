<script setup>
import DiscogsInput from './components/DiscogsInput.vue';
// import AlbumInfo from './components/AlbumInfo.vue';
// import ScratchCode from './components/ScratchCode.vue';
</script>

<template>
  <div class="bg-gray-100 min-h-screen">
    <header class="bg-[#1e1e1e] text-white shadow-md">
      <div class="container mx-auto px-4 py-4 flex justify-between items-center">
        <h1 class="text-2xl font-bold tracking-wider"> Atlas Record Store</h1>
      </div>
    </header>

    <main class="flex-1 container mx-auto px-4 py-8">
      <!-- Placeholder slot -->
      <section class="bg-white rounded-xl shadow-md p-6">
        <DiscogsInput />
      </section>
    </main>


    <footer class="bg-[#1e1e1e] text-white mt-10">
      <div class="container mx-auto px-4 py-6 flex flex-col md:flex-row justify-between items-center text-sm">
        <p>© 2025 Atlas Record Store. All rights reserved.</p>
        <div class="space-x-4 mt-2 md:mt-0">
          <a href="#" class="hover:text-yellow-300">Instagram</a>
          <a href="#" class="hover:text-yellow-300">Contact</a>
        </div>
      </div>
    </footer>
  </div>
</template>

<style scoped>
* {
  font-family: 'Space Grotesk', sans-serif;
}

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
