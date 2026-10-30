<script setup>
import { ref, computed } from 'vue'

// States bawaan
const tasks = ref([])
const inputTask = ref('')

// States tambahan untuk fitur baru
const inputCategory = ref('Penting')
const editingId = ref(null)
const editingTitle = ref('')

// Action bawaan: Tambah Task
const addTask = (title) => {
  if (!title.trim()) return

  const newTask = {
    id: new Date().getTime(),
    title: title,
    category: inputCategory.value,
    status: false
  }

  tasks.value.push(newTask)
  inputTask.value = ''
}

// Action bawaan: Hapus Task
const deleteTask = (id) => {
  const task = tasks.value.find((item) => item.id === id)

  if (!task) {
    alert('Task tidak ditemukan')
    return
  }

  const confirmation = confirm('Apakah anda yakin ingin menghapus?')

  if (confirmation) {
    tasks.value = tasks.value.filter((item) => item.id !== id)
  }
}

// Action tambahan: Edit Task
const startEdit = (task) => {
  editingId.value = task.id
  editingTitle.value = task.title
}

const saveEdit = (task) => {
  if (editingTitle.value.trim() !== '') {
    task.title = editingTitle.value
  }
  editingId.value = null
}

const cancelEdit = () => {
  editingId.value = null
}

// Computed Properties untuk Bento Grid Metrics & Quick Status Cards
const totalTasks = computed(() => tasks.value.length)
const completedTasks = computed(() => tasks.value.filter((t) => t.status).length)
const pendingTasks = computed(() => tasks.value.filter((t) => !t.status).length)
const completionRate = computed(() => {
  if (totalTasks.value === 0) return 0
  return Math.round((completedTasks.value / totalTasks.value) * 100)
})
</script>

<template>
  <main class="min-h-screen bg-gradient-to-br from-emerald-100 via-teal-50 to-blue-100 p-4 sm:p-8 text-slate-800 font-sans">
    <div class="mx-auto max-w-5xl space-y-6">
      
      <!-- HEADER & FORMULIR UTAMA (Atas) -->
      <header class="rounded-3xl border border-white/40 bg-white/30 backdrop-blur-md p-6 shadow-xl transition-all">
        <h1 class="mb-4 text-center text-3xl font-extrabold tracking-tight text-teal-900 drop-shadow-sm">
          Taskify Vue
        </h1>

        <!-- Form Tambah Tugas -->
        <form @submit.prevent="addTask(inputTask)" class="flex flex-col sm:flex-row gap-3">
          <input
            v-model="inputTask"
            type="text"
            placeholder="Masukkan tugas baru..."
            aria-label="Tugas baru"
            class="min-w-0 flex-1 rounded-2xl border border-white/50 bg-white/40 px-5 py-3.5 text-slate-800 placeholder-slate-400 outline-none backdrop-blur-sm transition focus:bg-white/60 focus:ring-2 focus:ring-teal-400"
          />

          <!-- Selector Kategori / Tag -->
          <select
            v-model="inputCategory"
            class="rounded-2xl border border-white/50 bg-white/40 px-4 py-3.5 text-slate-700 outline-none backdrop-blur-sm transition focus:bg-white/60 focus:ring-2 focus:ring-teal-400"
          >
            <option value="Penting">Penting</option>
            <option value="Kerja">Kerja</option>
            <option value="Pribadi">Pribadi</option>
            <option value="Belajar">Belajar</option>
          </select>

          <!-- Tombol Tambah dengan Efek Glass & Bold Color -->
          <button
            type="submit"
            class="rounded-2xl border border-teal-300/40 bg-teal-600/80 px-8 py-3.5 font-bold text-white shadow-lg backdrop-blur-md transition hover:bg-teal-600 active:scale-95"
          >
            Tambah
          </button>
        </form>
      </header>

      <!-- BENTO GRID LAYOUT -->
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        
        <!-- BENTO CARD 1: Ringkasan Produktivitas (Task Metrics) -->
        <div class="rounded-3xl border border-white/40 bg-white/30 backdrop-blur-md p-6 shadow-xl flex flex-col justify-between space-y-4">
          <h2 class="text-lg font-bold text-teal-900 flex items-center gap-2">
            📊 Ringkasan Produktivitas
          </h2>
          <div class="space-y-3">
            <div>
              <div class="flex justify-between text-sm font-semibold text-slate-600 mb-1">
                <span>Progress Selesai</span>
                <span>{{ completionRate }}%</span>
              </div>
              <div class="h-3 w-full rounded-full bg-white/50 overflow-hidden">
                <div
                  class="h-full bg-gradient-to-r from-teal-500 to-blue-500 transition-all duration-500"
                  :style="{ width: completionRate + '%' }"
                ></div>
              </div>
            </div>
            <p class="text-xs text-slate-600">
              Menampilkan tingkat penyelesaian dari seluruh daftar tugasmu.
            </p>
          </div>
        </div>

        <!-- BENTO CARD 2: Mode Tampilan Cepat (Quick Status Cards) -->
        <div class="md:col-span-2 grid grid-cols-3 gap-4">
          <div class="rounded-3xl border border-white/40 bg-emerald-200/30 backdrop-blur-md p-5 shadow-lg flex flex-col justify-center items-center text-center">
            <span class="text-3xl font-extrabold text-emerald-800">{{ totalTasks }}</span>
            <span class="text-xs font-semibold text-emerald-900 mt-1">Total Tugas</span>
          </div>

          <div class="rounded-3xl border border-white/40 bg-blue-200/30 backdrop-blur-md p-5 shadow-lg flex flex-col justify-center items-center text-center">
            <span class="text-3xl font-extrabold text-blue-800">{{ pendingTasks }}</span>
            <span class="text-xs font-semibold text-blue-900 mt-1">Belum Selesai</span>
          </div>

          <div class="rounded-3xl border border-white/40 bg-teal-200/30 backdrop-blur-md p-5 shadow-lg flex flex-col justify-center items-center text-center">
            <span class="text-3xl font-extrabold text-teal-800">{{ completedTasks }}</span>
            <span class="text-xs font-semibold text-teal-900 mt-1">Selesai</span>
          </div>
        </div>

        <!-- BENTO CARD 3: Daftar Tugas (Main Task List Container) -->
        <div class="md:col-span-3 rounded-3xl border border-white/40 bg-white/30 backdrop-blur-md p-6 shadow-xl">
          <h2 class="text-xl font-bold text-teal-900 mb-4">Daftar Tugas</h2>

          <!-- List Tasks -->
          <ul class="space-y-3">
            <li
              v-for="task in tasks"
              :key="task.id"
              class="group flex flex-col sm:flex-row sm:items-center justify-between gap-3 rounded-2xl border border-white/40 bg-white/40 p-4 shadow-sm backdrop-blur-sm transition hover:bg-white/60"
            >
              <!-- Konten Tugas & Kategori -->
              <div class="flex items-center gap-3 flex-1 min-w-0">
                <input
                  v-model="task.status"
                  type="checkbox"
                  class="h-5 w-5 accent-teal-600 rounded cursor-pointer"
                />

                <!-- Mode Edit In-place -->
                <div v-if="editingId === task.id" class="flex-1 flex gap-2">
                  <input
                    v-model="editingTitle"
                    type="text"
                    class="flex-1 rounded-xl border border-white/60 bg-white/70 px-3 py-1 text-sm outline-none"
                    @keyup.enter="saveEdit(task)"
                  />
                  <button
                    @click="saveEdit(task)"
                    class="rounded-xl bg-emerald-600/80 border border-emerald-400/40 px-3 py-1 text-xs font-bold text-white backdrop-blur-md"
                  >
                    Simpan
                  </button>
                  <button
                    @click="cancelEdit"
                    class="rounded-xl bg-slate-400/80 border border-slate-300/40 px-3 py-1 text-xs font-bold text-white backdrop-blur-md"
                  >
                    Batal
                  </button>
                </div>

                <!-- Mode Tampil Normal -->
                <div v-else class="flex items-center gap-2 flex-1 min-w-0">
                  <span
                    class="truncate font-medium text-slate-800"
                    :class="task.status ? 'line-through text-slate-400' : ''"
                  >
                    {{ task.title }}
                  </span>

                  <!-- Badge Kategori / Tag -->
                  <span
                    class="rounded-full bg-teal-100/70 border border-teal-300/50 px-2.5 py-0.5 text-xs font-semibold text-teal-800"
                  >
                    {{ task.category || 'Umum' }}
                  </span>
                </div>
              </div>

              <!-- Tombol Aksi (Edit & Hapus) -->
              <div class="flex items-center gap-2 self-end sm:self-center">
                <button
                  v-if="editingId !== task.id"
                  @click="startEdit(task)"
                  class="rounded-xl bg-blue-600/80 border border-blue-400/30 px-3 py-1.5 text-xs font-bold text-white backdrop-blur-md transition hover:bg-blue-600"
                >
                  Edit
                </button>
                <button
                  @click="deleteTask(task.id)"
                  class="rounded-xl bg-rose-600/80 border border-rose-400/30 px-3 py-1.5 text-xs font-bold text-white backdrop-blur-md transition hover:bg-rose-600"
                >
                  Hapus
                </button>
              </div>
            </li>

            <!-- Placeholder jika belum ada tugas -->
            <li
              v-if="tasks.length === 0"
              class="text-center py-8 text-slate-500 font-medium"
            >
              Belum ada tugas. Tambahkan tugas baru di atas!
            </li>
          </ul>
        </div>

      </div>
    </div>
  </main>
</template>