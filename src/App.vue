<template>
  <div class="min-h-screen bg-slate-50 text-slate-800 font-sans antialiased py-8 px-4 flex flex-col justify-center">
    <div class="max-w-4xl mx-auto w-full">

      <!-- State: Loading Global -->
      <div v-if="loading" class="flex flex-col items-center justify-center py-20 space-y-4">
        <div class="relative">
          <!-- Efek Glow Ringan di Belakang Spinner -->
          <div class="absolute inset-0 bg-emerald-100 rounded-full blur-xl opacity-60 scale-150 animate-pulse"></div>
          <div class="relative animate-spin rounded-full h-12 w-12 border-4 border-emerald-100 border-t-emerald-600">
          </div>
        </div>
        <div class="text-center space-y-1">
          <p class="text-sm font-bold text-slate-700">Memvalidasi Akses</p>
          <p class="text-xs text-slate-400">Mohon tunggu sebentar...</p>
        </div>
      </div>

      <!-- State: Error Global (Akses Ditolak / Halaman Tidak Ditemukan) -->
      <div v-else-if="error" class="w-full max-w-md mx-auto text-center space-y-8">

        <!-- Branding Atas -->
        <div class="space-y-2">
          <div
            class="inline-flex items-center justify-center bg-emerald-50 text-emerald-600 p-2.5 rounded-2xl border border-emerald-100/80">
            <!-- Ikon Masjid / Ka'bah Minimalis -->
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24"
              stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4" />
            </svg>
          </div>
          <h1 class="text-xs font-bold tracking-wider text-slate-400 uppercase">
            Sistem Rekap Tabungan Qurban Kelompok 3
          </h1>
        </div>

        <!-- Card Box Error -->
        <div class="bg-white border border-slate-100 rounded-3xl p-8 shadow-sm space-y-6">
          <!-- Ilustrasi Peringatan Minimalis -->
          <div class="flex justify-center">
            <div class="relative">
              <div class="absolute inset-0 bg-rose-100 rounded-full blur-xl opacity-40 scale-125"></div>
              <div class="relative bg-rose-50 text-rose-500 p-4 rounded-2xl border border-rose-100/80">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24"
                  stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                </svg>
              </div>
            </div>
          </div>

          <!-- Teks Peringatan -->
          <div class="space-y-2">
            <h2 class="text-base font-bold text-slate-800">Akses Tidak Dikenali</h2>
            <p class="text-xs text-slate-400 leading-relaxed max-w-xs mx-auto">
              {{ error }}
            </p>
          </div>

          <!-- Footer Bantuan -->
          <div class="pt-5 border-t border-slate-100">
            <p class="text-[10px] text-slate-400">
              Butuh bantuan? Hubungi Admin Kelompok 3 Qurban
            </p>
          </div>
        </div>

      </div>

      <!-- Render Halaman Dinamis -->
      <div v-else>
        <AdminDashboard v-if="currentPage === 'admin'" :api-url="API_URL" :list-bulan-props="listBulan" />
        <UserDetail v-else-if="currentPage === 'user'" :api-url="API_URL" :user-name="params.name"
          :user-token="params.token" :list-bulan-props="listBulan" />
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
          error.value = 'Akses ditolak. Token admin yang Anda masukkan tidak valid.'
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
          } else {
            error.value = 'Token atau data nama tidak cocok dengan database.'
          }
        } catch (e) {
          console.error("Gagal memetakan bulan secara dinamis")
          error.value = 'Gagal memuat data dari server. Pastikan koneksi internet Anda lancar.'
        }
      } else {
        error.value = 'Halaman tidak ditemukan. Pastikan Anda menggunakan tautan resmi yang sah.'
      }
      loading.value = false
    })

    return { currentPage, loading, error, API_URL, params, listBulan }
  }
}
</script>