<!-- App.vue -->
<template>
  <div class="task-management-app">
    <div class="app-container">
      <header class="app-header">
        <div class="header-content">
          <h1 class="app-title">{{ title }}</h1>
          <div class="theme-toggle">
            <button @click="toggleTheme" class="theme-btn">
              🌓 Theme
            </button>
          </div>
        </div>
        <div v-if="user" class="user-section">
          <span class="welcome-text">Welcome, {{ user.name }}</span>
          <button @click="logout" class="btn logout-btn">
            <span class="btn-icon">→</span> Logout
          </button>
        </div>
      </header>

      <!-- Login Form -->
      <div v-if="!user" class="login-container">
        <div class="login-form">
          <h2 class="login-title">Welcome Back</h2>
          <div class="input-group">
            <label>Username</label>
            <input 
              v-model="loginForm.username" 
              class="form-input"
              placeholder="Enter your username"
            />
          </div>
          <div class="input-group">
            <label>Password</label>
            <input 
              v-model="loginForm.password" 
              type="password" 
              class="form-input"
              placeholder="Enter your password"
            />
          </div>
          <button @click="login" class="btn login-btn">
            <span class="btn-icon">↪</span> Login
          </button>
        </div>
      </div>

      <div v-else class="main-content">
        <!-- Task Creation Section -->
        <div class="task-creation-section">
          <div class="task-form">
            <div class="form-header">
              <h3>Create New Task</h3>
              <button @click="toggleTaskForm" class="btn toggle-form-btn">
                {{ showTaskForm ? '− Close' : '+ New Task' }}
              </button>
            </div>
            
            <div v-if="showTaskForm" class="form-content">
              <div class="input-group">
                <label>Title</label>
                <input 
                  v-model="newTask.title" 
                  @keyup.enter="addTask"
                  class="form-input"
                  placeholder="What needs to be done?"
                />
              </div>

              <div class="input-group">
                <label>Description</label>
                <textarea 
                  v-model="newTask.description" 
                  class="form-textarea"
                  placeholder="Add more details..."
                ></textarea>
              </div>

              <div class="form-row">
                <div class="input-group">
                  <label>Priority</label>
                  <select v-model="newTask.priority" class="form-select">
                    <option value="low">Low</option>
                    <option value="medium">Medium</option>
                    <option value="high">High</option>
                  </select>
                </div>

                <div class="input-group">
                  <label>Category</label>
                  <select v-model="newTask.category" class="form-select">
                    <option value="work">Work</option>
                    <option value="personal">Personal</option>
                    <option value="shopping">Shopping</option>
                    <option value="health">Health</option>
                  </select>
                </div>

                <div class="input-group">
                  <label>Due Date</label>
                  <input 
                    type="date" 
                    v-model="newTask.dueDate"
                    class="form-input"
                  />
                </div>
              </div>

              <button @click="addTask" class="btn create-task-btn">
                <span class="btn-icon">+</span> Create Task
              </button>
            </div>
          </div>
        </div>

        <!-- Task Management Section -->
        <div class="task-management-section">
          <div class="management-header">
            <div class="search-bar">
              <input 
                v-model="searchQuery" 
                placeholder="Search tasks..."
                class="search-input"
              />
              <span class="search-icon">🔍</span>
            </div>

            <div class="filter-section">
              <div class="filter-group">
                <label>Status</label>
                <div class="btn-group">
                  <button 
                    v-for="filter in filters" 
                    :key="filter"
                    :class="['filter-btn', { active: currentFilter === filter }]"
                    @click="currentFilter = filter"
                  >
                    {{ filter }}
                  </button>
                </div>
              </div>

              <div class="filter-group">
                <label>Category</label>
                <div class="btn-group">
                  <button 
                    v-for="cat in categories" 
                    :key="cat"
                    :class="['filter-btn', { active: currentCategory === cat }]"
                    @click="currentCategory = cat"
                  >
                    {{ cat }}
                  </button>
                </div>
              </div>
            </div>
          </div>

          <!-- Tasks Display -->
          <div class="tasks-container">
            <!-- Urgent Tasks -->
            <div v-if="urgentTasks.length" class="urgent-tasks">
              <h3 class="section-title">
                <span class="urgent-icon">⚠️</span> Urgent Tasks
              </h3>
              <task-item
                v-for="task in urgentTasks"
                :key="task.id"
                :task="task"
                @update="updateTask"
                @delete="deleteTask"
              />
            </div>

            <!-- Regular Tasks -->
            <div class="regular-tasks">
              <h3 class="section-title">
                <span class="tasks-icon">📋</span> Tasks
                <span class="task-count">({{ filteredTasks.length }})</span>
              </h3>
              <div v-if="filteredTasks.length === 0" class="no-tasks">
                No tasks found
              </div>
              <task-item
                v-else
                v-for="task in filteredTasks"
                :key="task.id"
                :task="task"
                @update="updateTask"
                @delete="deleteTask"
              />
            </div>
          </div>

          <!-- Task Statistics -->
          <div class="task-statistics">
            <div class="stats-content">
              <div class="stats-item">
                <span class="stats-label">Total Tasks:</span>
                <span class="stats-value">{{ tasks.length }}</span>
              </div>
              <div class="stats-item">
                <span class="stats-label">Completed:</span>
                <span class="stats-value">{{ completedCount }}</span>
              </div>
              <div class="stats-item">
                <span class="stats-label">Urgent:</span>
                <span class="stats-value urgent">{{ urgentCount }}</span>
              </div>
            </div>
            <button 
              v-if="completedCount" 
              @click="clearCompleted"
              class="btn clear-btn"
            >
              Clear Completed
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import TaskItem from './components/TaskItem.vue'

export default {
  name: 'TaskManagementApp',
  components: {
    TaskItem
  },
  data() {
    return {
      title: 'Task Master Pro',
      theme: 'light',
      showTaskForm: false,
      user: null,
      loginForm: {
        username: '',
        password: ''
      },
      newTask: {
        title: '',
        description: '',
        priority: 'medium',
        category: 'work',
        dueDate: '',
      },
      tasks: [],
      searchQuery: '',
      currentFilter: 'All',
      currentCategory: 'All',
      filters: ['All', 'Active', 'Completed'],
      categories: ['All', 'Work', 'Personal', 'Shopping', 'Health']
    }
  },
  computed: {
    // ... (previous computed properties remain the same)
  },
  methods: {
    toggleTheme() {
      this.theme = this.theme === 'light' ? 'dark' : 'light'
      document.body.setAttribute('data-theme', this.theme)
    },
    toggleTaskForm() {
      this.showTaskForm = !this.showTaskForm
    },
    // ... (other methods remain the same)
  }
}
</script>

<style>
:root {
  --primary-color: #4A90E2;
  --success-color: #27AE60;
  --warning-color: #F39C12;
  --danger-color: #E74C3C;
  --text-color: #2C3E50;
  --bg-color: #F5F7FA;
  --card-bg: #FFFFFF;
  --border-color: #E1E8ED;
  --shadow-color: rgba(0, 0, 0, 0.1);
}

[data-theme="dark"] {
  --primary-color: #64B5F6;
  --success-color: #81C784;
  --warning-color: #FFB74D;
  --danger-color: #E57373;
  --text-color: #ECEFF1;
  --bg-color: #263238;
  --card-bg: #37474F;
  --border-color: #455A64;
  --shadow-color: rgba(0, 0, 0, 0.3);
}

body {
  margin: 0;
  padding: 0;
  background-color: var(--bg-color);
  color: var(--text-color);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  transition: background-color 0.3s ease;
}

.task-management-app {
  min-height: 100vh;
  padding: 20px;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
}

/* Header Styles */
.app-header {
  background: var(--card-bg);
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 2px 10px var(--shadow-color);
  margin-bottom: 20px;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.app-title {
  margin: 0;
  color: var(--primary-color);
  font-size: 2em;
}

.user-section {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-top: 10px;
}

/* Form Styles */
.input-group {
  margin-bottom: 15px;
}

.input-group label {
  display: block;
  margin-bottom: 5px;
  color: var(--text-color);
  font-weight: 500;
}

.form-input,
.form-textarea,
.form-select {
  width: 100%;
  padding: 10px;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  background: var(--card-bg);
  color: var(--text-color);
  transition: border-color 0.3s ease;
}

.form-input:focus,
.form-textarea:focus,
.form-select:focus {
  border-color: var(--primary-color);
  outline: none;
}

.form-textarea {
  min-height: 100px;
  resize: vertical;
}

.form-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

/* Button Styles */
.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.btn-icon {
  font-size: 1.2em;
}

.create-task-btn {
  background: var(--success-color);
  color: white;
}

.logout-btn {
  background: var(--danger-color);
  color: white;
}

.filter-btn {
  background: var(--card-bg);
  color: var(--text-color);
  border: 1px solid var(--border-color);
  padding: 8px 15px;
  border-radius: 6px;
}

.filter-btn.active {
  background: var(--primary-color);
  color: white;
  border-color: var(--primary-color);
}

/* Task List Styles */
.tasks-container {
  margin: 20px 0;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
  color: var(--text-color);
}

.urgent-tasks {
  background: rgb(231, 76, 60, 0.1);
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 20px;
}

.task-statistics {
  background: var(--card-bg);
  padding: 20px;
  border-radius: 12px;
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stats-content {
  display: flex;
  gap: 20px;
}

.stats-item {
  display: flex;
  flex-direction: column;
}

.stats-label {
  font-size: 0.9em;
  color: var(--text-color);
  opacity: 0.8;
}

.stats-value {
  font-size: 1.2em;
  font-weight: 600;
  color: var(--primary-color);
}

.stats-value.urgent {
  color: var(--danger-color);
}

/* Responsive Design */
@media (max-width: 768px) {
  .form-row {
    grid-template-columns: 1fr;
  }

  .task-statistics {
    flex-direction: column;
    gap: 15px;
  }

  .stats-content {
    width: 100%;
    justify-content: space-between;
  }
}
</style>
```

