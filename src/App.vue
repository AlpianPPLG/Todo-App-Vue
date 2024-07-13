<template>
  <div class="app">
    <div class="todo-list">
      <!-- Judul Todo List dengan fitur reload -->
      <h1 @click="reloadPage">Todo List</h1>

      <!-- Form untuk menambahkan atau mengedit task -->
      <form @submit.prevent="editing !== null ? updateTask() : addTask()" class="form-group">
        <input
          type="text"
          v-model="newTask"
          placeholder="Tambahkan task baru..."
          class="form-control"
        />
        <!-- Tombol Tambah atau Update -->
        <button type="submit" class="btn btn-primary" :disabled="loading">
          {{ editing !== null ? 'Update' : 'Tambah' }}
        </button>
        <!-- Tombol Batal saat sedang Edit -->
        <button
          v-if="editing !== null"
          @click="cancelEdit"
          class="btn btn-secondary"
          :disabled="loading"
        >
          Batal
        </button>
      </form>

      <!-- Daftar task -->
      <ul class="task-list">
        <li v-for="(task, index) in tasks" :key="index" class="task-item">
          <label>
            <!-- Checkbox untuk menandai task selesai -->
            <input type="checkbox" v-model="task.completed" class="form-check-input" />
            <!-- Teks task dengan penanda jika sudah selesai -->
            <span :class="{ completed: task.completed }">{{ task.title }}</span>
          </label>
          <!-- Aksi untuk Edit dan Hapus task -->
          <div class="task-actions">
            <button @click="editTask(index)" class="btn btn-warning" :disabled="loading">
              Edit
            </button>
            <button
              @click="removeTask(index)"
              class="btn btn-danger"
              style="margin-left: 5px"
              :disabled="loading"
            >
              Hapus
            </button>
          </div>
        </li>
      </ul>
    </div>

    <!-- Hak Cipta -->
    <footer class="footer">
      <!-- Teks hak cipta dengan tahun saat ini -->
      <p>&copy; {{ currentYear }} Alpian. All rights reserved.</p>
    </footer>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newTask: '',
      tasks: [],
      editing: null,
      loading: false // Variabel untuk menunjukkan status loading
    }
  },
  computed: {
    currentYear() {
      return new Date().getFullYear()
    }
  },
  methods: {
    // Fungsi untuk memuat ulang halaman
    reloadPage() {
      location.reload()
    },
    // Fungsi untuk menambahkan task baru
    addTask() {
      if (this.newTask.trim() !== '') {
        this.loading = true // Menandakan bahwa sedang loading
        setTimeout(() => {
          this.tasks.push({
            title: this.newTask.trim(),
            completed: false
          })
          this.newTask = ''
          this.loading = false // Selesai loading
        }, 1000) // Contoh delay 1 detik untuk simulasi loading
      } else {
        alert('Kolom input tidak boleh kosong!')
      }
    },
    // Fungsi untuk mengedit task yang ada
    editTask(index) {
      this.newTask = this.tasks[index].title
      this.editing = index
    },
    // Fungsi untuk menyimpan perubahan pada task yang sedang diedit
    updateTask() {
      if (this.newTask.trim() !== '') {
        this.loading = true // Menandakan bahwa sedang loading
        setTimeout(() => {
          this.tasks[this.editing].title = this.newTask.trim()
          this.newTask = ''
          this.editing = null
          this.loading = false // Selesai loading
        }, 1000) // Contoh delay 1 detik untuk simulasi loading
      } else {
        alert('Kolom input tidak boleh kosong!')
      }
    },
    // Fungsi untuk membatalkan pengeditan task
    cancelEdit() {
      this.newTask = ''
      this.editing = null
    },
    // Fungsi untuk menghapus task
    removeTask(index) {
      this.tasks.splice(index, 1)
      if (this.editing === index) {
        this.cancelEdit()
      }
    }
  }
}
</script>

<style>
.app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.todo-list {
  flex: 1; /* Mengisi ruang kosong */
  max-width: 600px;
  margin: auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.form-group {
  margin-bottom: 20px;
}

.form-control {
  padding: 8px;
  font-size: 16px;
  width: 60%;
  margin-right: 10px;
}

.btn {
  padding: 8px 16px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 4px;
}

.btn-primary {
  background-color: #007bff;
  color: white;
  border: none;
}

.btn-primary:hover {
  background-color: #0056b3;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
  border: none;
}

.btn-secondary:hover {
  background-color: #5a6268;
}

.btn-warning {
  background-color: #ffc107;
  color: black;
  border: none;
}

.btn-warning:hover {
  background-color: #e0a800;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
  border: none;
}

.btn-danger:hover {
  background-color: #bd2130;
}

.task-list {
  list-style-type: none;
  padding: 0;
}

.task-item {
  margin-bottom: 10px;
  display: flex;
  align-items: center;
}

.completed {
  text-decoration: line-through;
  color: #888;
}

.task-actions {
  margin-left: auto;
  display: flex;
  align-items: center;
}

h1 {
  cursor: pointer;
}

.footer {
  text-align: center;
  margin-top: auto; /* Mengatur footer di bagian bawah */
  color: black;
  padding: 10px;
  background-color: #f8f9fa;
}
</style>
