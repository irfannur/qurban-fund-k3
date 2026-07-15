<template>
  <div class="min-h-screen bg-slate-50 text-slate-800 font-sans antialiased py-8 px-4">
    <div class="max-w-4xl mx-auto">
      
      <!-- State: Loading Global -->
      <div v-if="loading" class="flex flex-col items-center justify-center pt-20 space-y-3">
        <div class="animate-spin rounded-full h-10 w-10 border-b-2 border-emerald-600"></div>
        <p class="text-sm text-slate-500 animate-pulse">Memvalidasi Akses...</p>
      </div>

      <!-- State: Error Global -->
      <div v-else-if="error" class="bg-red-50 border border-red-200 text-red-700 p-6 rounded-2xl text-center shadow-sm max-w-md mx-auto">
        <p class="font-semibold">{{ error }}</p>
      </div>

      <!-- Render Halaman Dinamis -->
      <div v-else>
        <AdminDashboard v-if="currentPage === 'admin'" :api-url="API_URL" :list-bulan-props="listBulan" />
        <UserDetail v-else-if="currentPage === 'user'" :api-url="API_URL" :user-name="params.name" :user-token="params.token" :list-bulan-props="listBulan" />
      </div>

    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import AdminDashboard from './components/AdminDashboard.vue'
import UserDetail from './components/UserDetail.vue'

export default {
  name: 'App',
  components: { AdminDashboard, UserDetail },
  setup() {
    // Membaca dari file .env
    const API_URL = import.meta.env.VITE_API_URL
    const TOKEN_ADMIN_RAHASIA = 'adminqurban2026'

    const currentPage = ref('')
    const loading = ref(true)
    const error = ref(null)
    const params = ref({ name: '', token: '' })
    const listBulan = ref([]) // Diisi secara dinamis dari API

    onMounted(async () => {
      const urlParams = new URLSearchParams(window.location.search)
      const pageParam = urlParams.get('page')
      const adminTokenParam = urlParams.get('admintoken')
      const nameParam = urlParams.get('name')
      const tokenParam = urlParams.get('token')

      if (pageParam === 'admin') {
        if (adminTokenParam === TOKEN_ADMIN_RAHASIA) {
          currentPage.value = 'admin'
        } else {
          error.value = 'Akses Ditolak. Token admin salah.'
        }
      } else if (nameParam && tokenParam) {
        params.value = { name: nameParam, token: tokenParam }
        currentPage.value = 'user'
        
        // Cek daftar bulan dari data user yang bersangkutan langsung
        try {
          const res = await fetch(`${API_URL}?name=${encodeURIComponent(nameParam)}&token=${tokenParam}`)
          const data = await res.json()
          if (!data.error) {
            listBulan.value = Object.keys(data.detail_bulanan)
          }
        } catch (e) {
          console.error("Gagal memetakan bulan secara dinamis")
        }
      } else {
        error.value = 'Halaman tidak ditemukan. Gunakan tautan yang sah.'
      }
      loading.value = false
    })

    return { currentPage, loading, error, API_URL, params, listBulan }
  }
}
</script>