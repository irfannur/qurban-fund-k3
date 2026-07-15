<template>
    <div class="space-y-6">
        <!-- Header Admin -->
        <div
            class="bg-gradient-to-br from-slate-800 to-slate-900 rounded-2xl p-6 text-white shadow-md flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
            <div>
                <p class="text-xs uppercase tracking-wider text-slate-400 font-semibold mb-1">Panel Kendali Admin</p>
                <h1 class="text-2xl font-bold">Rekap Tabungan Qurban</h1>
            </div>
            <!-- Filter Bulan Global -->
            <div class="bg-slate-700/50 p-1.5 rounded-xl border border-slate-700 w-full md:w-auto">
                <select v-model="selectedBulanGlobal"
                    class="bg-transparent text-white font-medium text-sm px-3 py-1.5 focus:outline-none w-full cursor-pointer">
                    <!-- Gunakan fungsi formatBulanIndo hanya untuk teks tampilan -->
                    <option v-for="bln in listBulan" :key="bln" :value="bln" class="text-slate-900">
                        {{ formatBulanIndo(bln) }}
                    </option>
                </select>
            </div>
        </div>

        <!-- STATS CARD & CHART SECTION -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-5">
            <!-- Card Total Saldo Akumulasi -->
            <div class="bg-white border border-slate-100 rounded-2xl p-6 shadow-sm flex flex-col justify-between">
                <div>
                    <span class="text-xs font-bold text-slate-400 uppercase tracking-wider">Total Tabungan Terkumpul</span>
                    <h3 class="text-2xl font-extrabold text-emerald-600 mt-2">{{ formatRupiah(totalKasSemuaBulan) }}
                    </h3>
                </div>
                <p class="text-[11px] text-slate-400 mt-4">
                    Akumulasi total tabungan dari seluruh anggota untuk semua bulan yang aktif.
                </p>
            </div>

            <!-- Card Chart Tren Saldo Bulanan -->
            <div class="md:col-span-2 bg-white border border-slate-100 rounded-2xl p-5 shadow-sm">
                <h3 class="text-sm font-bold text-slate-700 mb-3 flex items-center gap-1.5">
                    <span class="w-1.5 h-3.5 bg-emerald-500 rounded-full"></span>
                    Tren Pertumbuhan Kas Bulanan
                </h3>
                <div class="h-40">
                    <Line v-if="chartReady" :data="chartData" :options="chartOptions" />
                    <div v-else class="h-full flex items-center justify-center text-xs text-slate-400 animate-pulse">
                        Menyiapkan grafik...
                    </div>
                </div>
            </div>
        </div>

        <!-- Tabel Rekap Semua Orang -->
        <div class="bg-white border border-slate-100 rounded-2xl shadow-sm overflow-hidden">
            <div class="p-5 border-b border-slate-100">
                <h2 class="font-bold text-slate-700 flex items-center gap-2">
                    <span class="w-2 h-4 bg-emerald-500 rounded-full"></span>
                    Daftar Tabungan Anggota ({{ formatBulanIndo(selectedBulanGlobal) }})
                </h2>
            </div>

            <!-- Tambahan: Info Total Tabungan Khusus Bulan Terpilih -->
            <div class="px-5 py-4 bg-emerald-50/45 border-b border-slate-100 flex justify-between items-center text-sm">
                <span class="text-slate-600 font-semibold">Total Setoran Bulan Ini ({{
                    formatBulanIndo(selectedBulanGlobal) }}):</span>
                <span class="font-extrabold text-emerald-600 text-lg">{{ formatRupiah(totalKasBulanTerpilih) }}</span>
            </div>

            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr
                            class="bg-slate-50 text-slate-400 text-xs font-semibold uppercase border-b border-slate-100">
                            <th class="py-3 px-5">Nama Anggota</th>
                            <th class="py-3 px-5 text-right">Total Bulan Ini</th>
                            <th class="py-3 px-5 text-center">Aksi</th>
                        </tr>
                    </thead>
                    <tbody class="divide-y divide-slate-100 text-sm">
                        <tr v-for="user in rekapAdminData" :key="user.nama" class="hover:bg-slate-50/70 transition">
                            <td class="py-3.5 px-5 font-semibold text-slate-700">{{ user.nama }}</td>
                            <td class="py-3.5 px-5 text-right font-bold text-emerald-600">
                                {{ formatRupiah(user.detail_bulanan[selectedBulanGlobal]?.total_bulan || 0) }}
                            </td>
                            <td class="py-3.5 px-5 text-center flex items-center justify-center gap-2">
                                <!-- Tombol Lihat Detail (Aksi Baru) -->
                                <button @click="bukaDetailUser(user)"
                                    class="inline-flex items-center gap-1 bg-slate-100 hover:bg-slate-200 text-slate-700 font-semibold px-3 py-1.5 rounded-xl text-xs transition cursor-pointer">
                                    Detail
                                </button>
                                <!-- Tombol Bagikan Link -->
                                <button @click="shareLink(user.nama, user.token)"
                                    class="inline-flex items-center gap-1.5 bg-emerald-50 hover:bg-emerald-100 text-emerald-700 font-semibold px-3 py-1.5 rounded-xl text-xs transition cursor-pointer">
                                    <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none"
                                        viewBox="0 0 24 24" stroke="currentColor">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                            d="M8.684 10.742l4.632-2.316m0 0a3 3 0 102.217-4.217 3 3 0 00-2.217 4.217zm-4.632 6l4.632 2.316m0 0a3 3 0 102.217-4.217 3 3 0 00-2.217 4.217z" />
                                    </svg>
                                    Share
                                </button>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <!-- ---------------------------------------------------- -->
        <!-- DRAWER / SLIDE BOTTOM SHEET (DETAIL PER ANGGOTA)     -->
        <!-- ---------------------------------------------------- -->
        <div v-if="isDrawerOpen" class="fixed inset-0 z-50 overflow-hidden" role="dialog" aria-modal="true">
            <!-- Backdrop Gelap Belakang -->
            <div class="absolute inset-0 bg-slate-900/40 backdrop-blur-sm transition-opacity duration-300"
                @click="tutupDrawer"></div>

            <!-- Container Laci Bergerak -->
            <div
                class="absolute inset-x-0 bottom-0 max-h-[90vh] bg-slate-50 rounded-t-[2rem] shadow-2xl flex flex-col transition-transform transform duration-300 translate-y-0 overflow-hidden">

                <!-- Handle Drawer (Untuk visual slide) -->
                <div class="flex justify-center py-3 bg-white border-b border-slate-100 cursor-pointer"
                    @click="tutupDrawer">
                    <div class="w-12 h-1.5 bg-slate-300 rounded-full"></div>
                </div>

                <!-- Header Drawer -->
                <div class="px-6 py-4 bg-white border-b border-slate-100 flex justify-between items-center">
                    <div>
                        <span class="text-[10px] uppercase tracking-wider font-bold text-slate-400">Rincian Tabungan
                            Anggota</span>
                        <h3 class="text-lg font-bold text-slate-800">{{ selectedUserDetail?.nama }}</h3>
                    </div>
                    <!-- Tombol Close -->
                    <button @click="tutupDrawer"
                        class="p-2 hover:bg-slate-100 rounded-full text-slate-400 hover:text-slate-600 transition">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>
                </div>

                <!-- Konten di dalam Drawer (Scrollable) -->
                <div class="flex-1 overflow-y-auto p-6 space-y-6">
                    <!-- Total Akumulasi Individu -->
                    <div
                        class="bg-gradient-to-r from-emerald-600 to-teal-700 text-white rounded-2xl p-5 shadow-sm flex justify-between items-center">
                        <span class="text-xs font-semibold uppercase tracking-wider">Total Tabungan Terkumpul</span>
                        <span class="text-xl font-extrabold">{{ formatRupiah(selectedUserDetail?.total_semua_bulan || 0)
                        }}</span>
                    </div>

                    <!-- Rincian Tabel Mingguan dengan Filter Bulan -->
                    <div class="bg-white border border-slate-100 rounded-2xl shadow-sm overflow-hidden">
                        <!-- Header Kontrol Filter -->
                        <div
                            class="p-4 border-b border-slate-100 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3">
                            <h4 class="text-xs font-bold text-slate-700 flex items-center gap-1.5">
                                <span class="w-1.5 h-3 bg-emerald-500 rounded-full"></span>
                                Rincian Setoran Bulanan
                            </h4>
                            <!-- Dropdown Filter Bulan -->
                            <div class="bg-slate-100 p-1 rounded-lg border border-slate-200/60 w-full sm:w-auto">
                                <select v-model="drawerSelectedBulan"
                                    class="bg-transparent text-slate-700 font-semibold text-[11px] px-2 py-1 focus:outline-none w-full cursor-pointer">
                                    <option v-for="bln in listBulan" :key="bln" :value="bln">
                                        {{ formatBulanIndo(bln) }}
                                    </option>
                                </select>
                            </div>
                        </div>

                        <!-- Total Per Bulan Terpilih di Drawer -->
                        <div
                            class="px-4 py-3 bg-emerald-50/40 border-b border-slate-100 flex justify-between items-center text-xs">
                            <span class="text-slate-500 font-semibold">Total Bulan Ini ({{
                                formatBulanIndo(drawerSelectedBulan)
                                }}):</span>
                            <span class="font-bold text-emerald-600">{{ formatRupiah(totalBulanTerpilihDrawer) }}</span>
                        </div>

                        <!-- Tabel Mingguan -->
                        <table class="w-full text-left border-collapse">
                            <thead>
                                <tr
                                    class="bg-slate-50/50 text-slate-400 text-[10px] font-bold uppercase border-b border-slate-100">
                                    <th class="py-2.5 px-4">Minggu Ke-</th>
                                    <th class="py-2.5 px-4">Tanggal Setor</th>
                                    <th class="py-2.5 px-4 text-right">Nominal</th>
                                </tr>
                            </thead>
                            <tbody class="divide-y divide-slate-50 text-xs">
                                <tr v-for="setor in setoranBulanTerpilihDrawer" :key="setor.minggu"
                                    class="hover:bg-slate-50/30">
                                    <td class="py-2.5 px-4 font-semibold text-slate-500">Minggu {{ setor.minggu }}</td>
                                    <td class="py-2.5 px-4 text-slate-600">{{ setor.tanggal }}</td>
                                    <td class="py-2.5 px-4 text-right font-bold text-slate-700">{{
                                        formatRupiah(setor.nominal) }}</td>
                                </tr>
                                <tr v-if="setoranBulanTerpilihDrawer.length === 0">
                                    <td colspan="3" class="py-4 text-center text-slate-400 italic">Belum ada catatan
                                        setoran.</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
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
    name: 'AdminDashboard',
    components: { Line },
    props: {
        apiUrl: String,
        listBulanProps: Array
    },
    setup(props) {
        const rekapAdminData = ref([])
        const listBulan = ref([])
        const selectedBulanGlobal = ref('')
        const chartReady = ref(false)

        // State untuk Drawer Detail Anggota
        const isDrawerOpen = ref(false)
        const selectedUserDetail = ref(null)
        const drawerSelectedBulan = ref('')

        const formatRupiah = (angka) => {
            return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(angka)
        }

        const formatBulanIndo = (bulanEN) => {
            if (!bulanEN) return '';

            // Kamus terjemahan bulan Inggris -> Indonesia
            const kamusBulan = {
                'january': 'Januari',
                'february': 'Februari',
                'march': 'Maret',
                'april': 'April',
                'may': 'Mei',
                'june': 'Juni',
                'july': 'Juli',
                'august': 'Agustus',
                'september': 'September',
                'october': 'Oktober',
                'november': 'November',
                'december': 'Desember'
            };

            // Pisahkan nama bulan dan tahun (Contoh: "July 2026" -> ["July", "2026"])
            const bagian = bulanEN.split(' ');
            if (bagian.length < 2) return bulanEN;

            const namaBulanEN = bagian[0].toLowerCase();
            const tahun = bagian[1];

            const namaBulanID = kamusBulan[namaBulanEN] || bagian[0];
            return `${namaBulanID} ${tahun}`;
        };

        // Hitung total kas keseluruhan (Semua bulan)
        const totalKasSemuaBulan = computed(() => {
            return rekapAdminData.value.reduce((acc, user) => acc + (user.total_semua_bulan || 0), 0)
        })

        // 1. HITUNG TOTAL KAS KHUSUS BULAN YANG DIPILIH SAJA (UNTUK TAMPILAN DI ATAS TABEL)
        const totalKasBulanTerpilih = computed(() => {
            if (!selectedBulanGlobal.value) return 0
            return rekapAdminData.value.reduce((acc, user) => {
                const totalBulanIni = user.detail_bulanan[selectedBulanGlobal.value]?.total_bulan || 0
                return acc + totalBulanIni
            }, 0)
        })

        // Hitung total kas per bulan (untuk chart)
        const totalKasPerBulan = computed(() => {
            const dataBulan = {}
            listBulan.value.forEach(bulan => {
                dataBulan[bulan] = 0
                rekapAdminData.value.forEach(user => {
                    if (user.detail_bulanan && user.detail_bulanan[bulan]) {
                        dataBulan[bulan] += user.detail_bulanan[bulan].total_bulan || 0
                    }
                })
            })
            return dataBulan
        })

        // Logika Pengambilan Data Untuk Drawer
        const setoranBulanTerpilihDrawer = computed(() => {
            if (!selectedUserDetail.value || !drawerSelectedBulan.value) return []
            const bulanData = selectedUserDetail.value.detail_bulanan[drawerSelectedBulan.value]
            return bulanData ? (bulanData.setoran || []) : []
        })

        const totalBulanTerpilihDrawer = computed(() => {
            if (!selectedUserDetail.value || !drawerSelectedBulan.value) return 0
            const bulanData = selectedUserDetail.value.detail_bulanan[drawerSelectedBulan.value]
            return bulanData ? (bulanData.total_bulan || 0) : 0
        })

        // Buka & Tutup Drawer
        const bukaDetailUser = (user) => {
            selectedUserDetail.value = user
            drawerSelectedBulan.value = selectedBulanGlobal.value // Samakan filter awal dengan filter dashboard
            isDrawerOpen.value = true
        }

        const tutupDrawer = () => {
            isDrawerOpen.value = false
            selectedUserDetail.value = null
        }

        // Chart Setup
        const chartData = computed(() => {
            return {
                labels: listBulan.value.map(b => formatBulanIndo(b)),
                datasets: [
                    {
                        label: 'Total Tabungan Bulanan',
                        backgroundColor: 'rgba(16, 185, 129, 0.1)',
                        borderColor: '#10b981',
                        borderWidth: 2.5,
                        pointBackgroundColor: '#10b981',
                        pointHoverRadius: 6,
                        tension: 0.35,
                        fill: true,
                        data: listBulan.value.map(bulan => totalKasPerBulan.value[bulan] || 0)
                    }
                ]
            }
        })

        const chartOptions = {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
                legend: { display: false },
                tooltip: {
                    callbacks: {
                        label: function (context) {
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
                        callback: function (value) {
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

        const shareLink = (nama, token) => {
            const baseUrl = window.location.origin + window.location.pathname
            const fullShareUrl = `${baseUrl}?name=${encodeURIComponent(nama)}&token=${token}`
            navigator.clipboard.writeText(fullShareUrl).then(() => {
                alert(`Link rekap untuk ${nama} berhasil disalin!`)
            })
        }

        onMounted(async () => {
            try {
                const responseKeys = await fetch(`${props.apiUrl}?action=getKeys`, {
                    method: 'GET',
                    redirect: 'follow'
                })
                const dataKeys = await responseKeys.json()

                listBulan.value = dataKeys.bulan

                // --- LOGIKA BULAN SAAT INI ---
                const daftarNamaBulanEN = [
                    "January", "February", "March", "April", "May", "June",
                    "July", "August", "September", "October", "November", "December"
                ];
                const sekarang = new Date();
                const bulanTahunSekarang = `${daftarNamaBulanEN[sekarang.getMonth()]} ${sekarang.getFullYear()}`;

                const indexBulanSekarang = listBulan.value.findIndex(
                    b => b.toLowerCase().trim() === bulanTahunSekarang.toLowerCase().trim()
                );

                if (indexBulanSekarang !== -1) {
                    selectedBulanGlobal.value = listBulan.value[indexBulanSekarang];
                } else if (listBulan.value.length > 0) {
                    selectedBulanGlobal.value = listBulan.value[0];
                }

                const promises = dataKeys.users.map(async (anggota) => {
                    const res = await fetch(`${props.apiUrl}?name=${encodeURIComponent(anggota.nama)}&token=${anggota.token}`, {
                        method: 'GET',
                        redirect: 'follow'
                    })
                    const json = await res.json()
                    return { ...json, token: anggota.token }
                })

                rekapAdminData.value = await Promise.all(promises)
                chartReady.value = true
            } catch (err) {
                console.error("Gagal memuat data admin:", err)
            }
        })

        return {
            rekapAdminData,
            listBulan,
            selectedBulanGlobal,
            formatRupiah,
            formatBulanIndo,
            shareLink,
            totalKasSemuaBulan,
            totalKasBulanTerpilih, // <-- Diexport ke template
            chartData,
            chartOptions,
            chartReady,

            // Export Drawer
            isDrawerOpen,
            selectedUserDetail,
            drawerSelectedBulan,
            bukaDetailUser,
            tutupDrawer,
            setoranBulanTerpilihDrawer,
            totalBulanTerpilihDrawer
        }
    }
}
</script>