<template>
  <div class="bg-gray-100 flex items-center justify-center h-screen">
    <div class="bg-white rounded-lg shadow-xl p-8 max-w-md w-full text-center">
      <h1 class="text-4xl font-bold text-red-500 mb-4">404</h1>
      <h2 class="text-2xl font-semibold text-gray-800 mb-2">Halaman Tidak Ditemukan</h2>
      <p class="text-gray-600 mb-6">Maaf, halaman yang Anda cari tidak dapat ditemukan.</p>
      <router-link to="/" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
        Kembali ke Beranda
      </router-link>
    </div>
  </div>
</template>

<style scoped>
.bg-gray-100 {
  background-color: #f7fafc; /* Tailwind's bg-gray-100 */
}
.bg-white {
  background-color: #ffffff; /* Tailwind's bg-white */
}
.text-gray-800{
    color: #1e293b;
}
.text-gray-600 {
  color: #4b5563; /* Tailwind's text-gray-600 */
}
.text-red-500 {
  color: #ef4444; /* Tailwind's text-red-500 */
}
.text-4xl {
  font-size: 2.25rem; /* Tailwind's text-4xl */
  line-height: 2.5rem; /* Tailwind's leading-7 */
}
.font-bold {
  font-weight: 700; /* Tailwind's font-bold */
}
.text-2xl {
  font-size: 1.5rem; /* Tailwind's text-2xl */
  line-height: 2rem; /* Tailwind's leading-6 */
}
.font-semibold {
  font-weight: 600; /* Tailwind's font-semibold */
}
.mb-2 {
  margin-bottom: 0.5rem; /* Tailwind's mb-2 */
}
.mb-4 {
  margin-bottom: 1rem; /* Tailwind's mb-4 */
}
.mb-6 {
  margin-bottom: 1.5rem; /* Tailwind's mb-6 */
}
.rounded-lg {
  border-radius: 0.5rem; /* Tailwind's rounded-lg */
}
.shadow-xl {
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.05); /* Tailwind's shadow-xl */
}
.p-8 {
  padding: 2rem; /* Tailwind's p-8 */
}
.max-w-md {
  max-width: 28rem; /* Tailwind's max-w-md */
}
.w-full {
  width: 100%; /* Tailwind's w-full */
}
.text-center {
  text-align: center; /* Tailwind's text-center */
}
.flex {
  display: flex; /* Tailwind's flex */
}
.items-center {
  align-items: center; /* Tailwind's items-center */
}
.justify-center {
  justify-content: center; /* Tailwind's justify-center */
}
.h-screen {
  height: 100vh; /* Tailwind's h-screen */
}
.bg-blue-500 {
    background-color: #3b82f6;
}
.hover\:bg-blue-700:hover {
    background-color: #2563eb;
}
.text-white{
    color: #fff;
}
.py-2{
    padding-top: 0.5rem;
    padding-bottom: 0.5rem;
}
.px-4{
    padding-left: 1rem;
    padding-right: 1rem;
}
.rounded{
    border-radius: 0.375rem;
}
.focus\:outline-none:focus{
    outline: none;
}
.focus\:shadow-outline:focus {
    box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.5);
}
</style>