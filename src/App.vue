<script setup>
//import DiscogsInput from './components/DiscogsInput.vue';
import InputDiscogs from './components/InputDiscogs.vue';
//import NotFound from './components/NotFound.vue';
</script>

<template>
  <div class="bg-gray-100 min-h-screen">
    <header class="bg-[#1e1e1e] text-white shadow-md">
      <div class="container mx-auto px-4 py-4 flex justify-between items-center">
        <h1 class="text-2xl font-bold tracking-wider"> Counter Data</h1>
      </div>
    </header>

    <main class="flex-1 container mx-auto px-4 py-8">
      <!-- Placeholder slot -->
      <section>
        <DiscogsInput />
        <InputDiscogs />
        <NotFound />
      </section>
    </main>


    <footer class="bg-[#1e1e1e] text-white mt-10">
      <div class="container mx-auto px-4 py-6 flex flex-col md:flex-row justify-between items-center text-sm">
        <p>© 2025 Dhan & Tya. All rights reserved.</p>
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
