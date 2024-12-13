<template>
  <div :class="['app', theme]">
    <div class="todo-list">
      <h1 @click="reloadPage">Todo List</h1>

      <form @submit.prevent="editing !== null ? updateTask() : addTask()" class="form-group">
        <input
          type="text"
          v-model="newTask"
          placeholder="Tambahkan task baru..."
          class="form-control"
        />
        <input type="date" v-model="dueDate" class="form-control" :disabled="loading" />
        <select v-model="taskGroup" class="form-control" :disabled="loading">
          <option value="">Pilih Grup</option>
          <option value="work">Pekerjaan</option>
          <option value="personal">Pribadi</option>
          <option value="other">Lainnya</option>
        </select>
        <button type="submit" class="btn btn-primary" :disabled="loading">
          {{ editing !== null ? 'Update' : 'Tambah' }}
        </button>
        <button
          v-if="editing !== null"
          @click="cancelEdit"
          class="btn btn-secondary"
          :disabled="loading"
        >
          Batal
        </button>
      </form>

      <select v-model="filter" class="form-control" @change="applyFilter">
        <option value="all">Semua</option>
        <option value="completed">Selesai</option>
        <option value="active">Aktif</option>
      </select>

      <button @click="toggleTheme" class="btn btn-secondary">
        {{ theme === 'dark' ? 'Tema Terang' : 'Tema Gelap' }}
      </button>

      <ul class="task-list">
        <li v-for="(task, index) in filteredTasks" :key="index" class="task-item">
          <label>
            <input type="checkbox" v-model="task.completed" class="form-check-input" />
            <span :class="{ completed: task.completed }">
              {{ task.title }} (Due: {{ task.dueDate }}) [{{ task.group }}]
            </span>
          </label>
          <div class="task-actions">
            <button @click="editTask(index)" class="btn btn-warning" :disabled="loading">
              Edit
            </button>
            <button
              @click="confirmRemoveTask(index)"
              class="btn btn-danger"
              style="margin-left: 5px"
              :disabled="loading"
            >
              Hapus
            </button>
          </div>
          <div class="subtask-form" v-if="editing === index">
            <input
              type="text"
              v-model="newSubtask"
              placeholder="Tambahkan subtask..."
              class="form-control"
            />
            <button @click="addSubtask(index)" class="btn btn-primary" :disabled="loading">
              Tambah Subtask
            </button>
          </div>
          <ul class="subtask-list">
            <li v-for="(subtask, subIndex) in task.subtasks" :key="subIndex" class="subtask-item">
              <input type="checkbox" v-model="subtask.completed" class="form-check-input" />
              <span :class="{ completed: subtask.completed }">{{ subtask.title }}</span>
              <button
                @click="confirmRemoveSubtask(index, subIndex)"
                class="btn btn-danger btn-sm"
                style="margin-left: 5px"
              >
                Hapus
              </button>
            </li>
          </ul>
        </li>
      </ul>
    </div>

    <footer class="footer">
      <p>&copy; {{ currentYear }} Alpian. All rights reserved.</p>
    </footer>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newTask: '',
      dueDate: '',
      taskGroup: '',
      tasks: [],
      newSubtask: '',
      editing: null,
      loading: false,
      filter: 'all',
      theme: 'light'
    }
  },
  computed: {
    currentYear() {
      return new Date().getFullYear()
    },
    filteredTasks() {
      let tasksToShow = this.tasks
      if (this.filter === 'completed') {
        tasksToShow = tasksToShow.filter((task) => task.completed)
      } else if (this.filter === 'active') {
        tasksToShow = tasksToShow.filter((task) => !task.completed)
      }
      return tasksToShow
    }
  },
  methods: {
    reloadPage() {
      location.reload()
    },
    addTask() {
      if (this.newTask.trim() !== '' && this.taskGroup) {
        this.loading = true
        setTimeout(() => {
          this.tasks.push({
            title: this.newTask.trim(),
            completed: false,
            dueDate: this.dueDate,
            group: this.taskGroup,
            subtasks: []
          })
          this.saveTasks()
          this.newTask = ''
          this.dueDate = ''
          this.taskGroup = ''
          this.loading = false
        }, 1000)
      } else {
        alert('Kolom input tidak boleh kosong dan grup harus dipilih!')
      }
    },
    editTask(index) {
      this.newTask = this.tasks[index].title
      this.dueDate = this.tasks[index].dueDate
      this.taskGroup = this.tasks[index].group
      this.editing = index
      this.newSubtask = '' // Reset subtask input
    },
    updateTask() {
      if (this.newTask.trim() !== '' && this.taskGroup) {
        this.loading = true
        setTimeout(() => {
          this.tasks[this.editing].title = this.newTask.trim()
          this.tasks[this.editing].dueDate = this.dueDate
          this.tasks[this.editing].group = this.taskGroup
          this.saveTasks()
          this.newTask = ''
          this.dueDate = ''
          this.taskGroup = ''
          this.editing = null
          this.loading = false
        }, 1000)
      } else {
        alert('Kolom input tidak boleh kosong dan grup harus dipilih!')
      }
    },
    cancelEdit() {
      this.newTask = ''
      this.dueDate = ''
      this.taskGroup = ''
      this.editing = null
    },
    confirmRemoveTask(index) {
      if (confirm('Apakah Anda yakin ingin menghapus task ini?')) {
        this.removeTask(index)
      }
    },
    removeTask(index) {
      this.tasks.splice(index, 1)
      this.saveTasks()
      if (this.editing === index) {
        this.cancelEdit()
      }
    },
    addSubtask(taskIndex) {
      if (this.newSubtask.trim() !== '') {
        this.tasks[taskIndex].subtasks.push({
          title: this.newSubtask.trim(),
          completed: false
        })
        this.saveTasks()
        this.newSubtask = '' // Reset input subtask
      } else {
        alert('Kolom input subtask tidak boleh kosong!')
      }
    },
    confirmRemoveSubtask(taskIndex, subIndex) {
      if (confirm('Apakah Anda yakin ingin menghapus subtask ini?')) {
        this.removeSubtask(taskIndex, subIndex)
      }
    },
    removeSubtask(taskIndex, subIndex) {
      this.tasks[taskIndex].subtasks.splice(subIndex, 1)
      this.saveTasks()
    },
    saveTasks() {
      localStorage.setItem('tasks', JSON.stringify(this.tasks))
    },
    loadTasks() {
      const savedTasks = localStorage.getItem('tasks')
      if (savedTasks) {
        this.tasks = JSON.parse(savedTasks)
      }
    },
    toggleTheme() {
      this.theme = this.theme === 'light' ? 'dark' : 'light'
    }
  },
  created() {
    this.loadTasks()
  }
}
</script>

<style>
.app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  transition: background 0.3s ease;
}

.todo-list {
  flex: 1;
  max-width: 600px;
  margin: auto;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  background-color: #ffffff;
  transition: all 0.3s ease;
}

h1 {
  font-size: 2rem;
  text-align: center;
  cursor: pointer;
  color: #333;
  transition: color 0.3s;
}

.form-group {
  margin-bottom: 20px;
}

.form-control {
  padding: 10px;
  font-size: 16px;
  width: calc(100% - 22px);
  margin-bottom: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
  transition: border-color 0.3s;
}

.form-control:focus {
  border-color: #007bff;
  outline: none;
}

.btn {
  padding: 10px 16px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 4px;
  border: none;
  transition: background-color 0.3s;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-primary:hover {
  background-color: #0056b3;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #5a6268;
}

.btn-warning {
  background-color: #ffc107;
  color: black;
}

.btn-warning:hover {
  background-color: #e0a800;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn-danger:hover {
  background-color: #bd2130;
}

.task-list {
  list-style-type: none;
  padding: 0;
}

.task-item {
  margin-bottom: 15px;
  display: flex;
  flex-direction: column; /* Change to column for subtasks */
  background-color: #f8f9fa;
  padding: 10px;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.task-item:hover {
  background-color: #e2e6ea;
}

.completed {
  text-decoration: line-through;
  color: #888;
}

.task-actions {
  display: flex;
  align-items: center;
}

.subtask-form {
  margin-top: 10px;
}

.subtask-list {
  list-style-type: none;
  padding: 0;
  margin-top: 10px;
}

.subtask-item {
  display: flex;
  align-items: center;
  margin: 5px 0;
}

.footer {
  text-align: center;
  margin-top: auto;
  padding: 10px;
  background-color: #f8f9fa;
  border-top: 1px solid #ddd;
}

.dark {
  background-color: #343a40;
  color: white;
}

.dark .todo-list {
  background-color: #495057;
}

.dark h1 {
  color: #f8f9fa;
}

.dark .form-control {
  background-color: #495057;
  border-color: #6c757d;
  color: white;
}

.dark .form-control:focus {
  border-color: #007bff;
}

.dark .btn-primary {
  background-color: #007bff;
}

.dark .btn-secondary {
  background-color: #6c757d;
}

.dark .task-item {
  background-color: #6c757d;
}

.dark .task-item:hover {
  background-color: #5a6268;
}

.dark .footer {
  background-color: #495057;
  border-top: 1px solid #6c757d;
}
</style>
