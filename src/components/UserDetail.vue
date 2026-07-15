<template>
  <div class="space-y-6">
    <!-- Header Welcome User -->
    <div class="bg-gradient-to-br from-emerald-600 to-teal-700 rounded-2xl p-6 text-white shadow-md flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
      <div>
        <p class="text-xs uppercase tracking-wider text-emerald-100 font-semibold mb-1">Log Tabungan Anggota</p>
        <h1 class="text-2xl font-bold">Halo, {{ userData.nama || 'Anggota' }}!</h1>
      </div>
      <!-- Statistik Ringkas Mandiri -->
      <div class="bg-white/10 px-4 py-2 rounded-xl backdrop-blur-sm border border-white/10">
        <span class="text-[10px] text-emerald-100 uppercase block font-semibold">Total Tabungan Anda</span>
        <span class="text-lg font-extrabold">{{ formatRupiah(userData.total_semua_bulan || 0) }}</span>
      </div>
    </div>

    <!-- ---------------------------------------------------- -->
    <!-- CHART TREN TABUNGAN PRIBADI                          -->
    <!-- ---------------------------------------------------- -->
    <div class="bg-white border border-slate-100 rounded-2xl p-5 shadow-sm">
      <h3 class="text-sm font-bold text-slate-700 mb-3 flex items-center gap-1.5">
        <span class="w-1.5 h-3.5 bg-emerald-500 rounded-full"></span>
        Tren Tabungan Bulanan Anda
      </h3>
      <!-- Container Chart -->
      <div class="h-44">
        <Line v-if="chartReady" :data="chartData" :options="chartOptions" />
        <div v-else class="h-full flex items-center justify-center text-xs text-slate-400 animate-pulse">
          Menyiapkan grafik ...
        </div>
      </div>
    </div>

    <!-- ---------------------------------------------------- -->
    <!-- TABEL DETAIL BULANAN (SINGLE TABLE DENGAN FILTER)    -->
    <!-- ---------------------------------------------------- -->
    <div class="bg-white border border-slate-100 rounded-2xl shadow-sm overflow-hidden">
      <!-- Header Tabel & Filter Dropdown -->
      <div class="p-5 border-b border-slate-100 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
        <h3 class="text-sm font-bold text-slate-700 flex items-center gap-2">
          <span class="w-2 h-4 bg-emerald-500 rounded-full"></span>
          Rincian Setoran Bulanan
        </h3>
        
        <!-- Dropdown Filter Bulan -->
        <div class="bg-slate-100 p-1 rounded-xl border border-slate-200/60 w-full sm:w-auto">
          <select v-model="selectedBulan" class="bg-transparent text-slate-700 font-semibold text-xs px-3 py-1.5 focus:outline-none w-full cursor-pointer">
            <option v-for="bln in listBulan" :key="bln" :value="bln">{{ bln }}</option>
          </select>
        </div>
      </div>

      <!-- Info Ringkas Total Bulan Terpilih -->
      <div class="px-5 py-4 bg-emerald-50/40 border-b border-slate-100 flex justify-between items-center text-sm">
        <span class="text-slate-500 font-medium">Total Terkumpul ({{ selectedBulan }}):</span>
        <span class="font-extrabold text-emerald-600 text-base">{{ formatRupiah(totalBulanTerpilih) }}</span>
      </div>

      <!-- Tabel Setoran Mingguan -->
      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse">
          <thead>
            <tr class="bg-slate-50/50 text-slate-400 text-[10px] font-bold uppercase border-b border-slate-100">
              <th class="py-3 px-5">Minggu Ke-</th>
              <th class="py-3 px-5">Tanggal Setor</th>
              <th class="py-3 px-5 text-right">Nominal</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-50 text-xs">
            <tr v-for="setor in setoranBulanTerpilih" :key="setor.minggu" class="hover:bg-slate-50/30">
              <td class="py-3 px-5 font-semibold text-slate-500">Minggu {{ setor.minggu }}</td>
              <td class="py-3 px-5 text-slate-600">{{ setor.tanggal }}</td>
              <td class="py-3 px-5 text-right font-bold text-slate-700">{{ formatRupiah(setor.nominal) }}</td>
            </tr>
            <tr v-if="setoranBulanTerpilih.length === 0">
              <td colspan="3" class="py-6 text-center text-slate-400 italic">
                Belum ada catatan setoran untuk bulan {{ selectedBulan }}.
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  Filler
} from 'chart.js'

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  Filler
)

export default {
  name: 'UserDetail',
  components: { Line },
  props: {
    apiUrl: String,
    userName: String,   
    userToken: String   
  },
  setup(props) {
    const userData = ref({ detail_bulanan: {} })
    const selectedBulan = ref('')
    const chartReady = ref(false)

    const formatRupiah = (angka) => {
      return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(angka)
    }

    // 1. DAFTAR BULAN SECARA DINAMIS DARI DATA USER YANG DITERIMA
    const listBulan = computed(() => {
      return Object.keys(userData.value.detail_bulanan || {})
    })

    // 2. AMBIL DATA SETORAN BULANAN BERDASARKAN FILTER YANG DIPILIH
    const setoranBulanTerpilih = computed(() => {
      if (!selectedBulan.value || !userData.value.detail_bulanan) return []
      const bulanData = userData.value.detail_bulanan[selectedBulan.value]
      return bulanData ? (bulanData.setoran || []) : []
    })

    // 3. HITUNG TOTAL KAS DARI BULAN TERPILIH
    const totalBulanTerpilih = computed(() => {
      if (!selectedBulan.value || !userData.value.detail_bulanan) return 0
      const bulanData = userData.value.detail_bulanan[selectedBulan.value]
      return bulanData ? (bulanData.total_bulan || 0) : 0
    })

    // 4. CONFIG DATA CHART UNTUK TREN BULANAN
    const chartData = computed(() => {
      return {
        labels: listBulan.value,
        datasets: [
          {
            label: 'Setoran Anda',
            backgroundColor: 'rgba(16, 185, 129, 0.1)', 
            borderColor: '#10b981', 
            borderWidth: 2.5,
            pointBackgroundColor: '#10b981',
            pointHoverRadius: 6,
            tension: 0.35, 
            fill: true,
            data: listBulan.value.map(bulan => userData.value.detail_bulanan[bulan]?.total_bulan || 0)
          }
        ]
      }
    })

    // 5. CONFIG TAMPILAN CHART
    const chartOptions = {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: {
          callbacks: {
            label: function(context) {
              let label = context.dataset.label || '';
              if (label) label += ': ';
              if (context.parsed.y !== null) {
                label += new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(context.parsed.y);
              }
              return label;
            }
          }
        }
      },
      scales: {
        y: {
          grid: { color: '#f1f5f9' },
          ticks: {
            font: { size: 10 },
            callback: function(value) {
              return value >= 1000000 ? (value / 1000000) + 'jt' : value;
            }
          }
        },
        x: {
          grid: { display: false },
          ticks: { font: { size: 10 } }
        }
      }
    }

    onMounted(async () => {
      if (!props.userName || !props.userToken) return

      try {
        const res = await fetch(`${props.apiUrl}?name=${encodeURIComponent(props.userName)}&token=${props.userToken}`, {
          method: 'GET',
          redirect: 'follow'
        })
        const json = await res.json()
        
        if (!json.error) {
          userData.value = json
          
          // Set default filter ke bulan pertama yang tersedia
          if (listBulan.value.length > 0) {
            selectedBulan.value = listBulan.value[0]
          }
          
          chartReady.value = true 
        } else {
          console.error("Error data:", json.error)
        }
      } catch (err) {
        console.error("Gagal mengambil detail data user:", err)
      }
    })

    return {
      userData,
      selectedBulan,
      listBulan,
      setoranBulanTerpilih,
      totalBulanTerpilih,
      formatRupiah,
      chartData,
      chartOptions,
      chartReady
    }
  }
}
</script>