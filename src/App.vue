<!-- App.vue -->
<template>
  <div class="todo-app">
    <h1>{{ title }}</h1>
    
    <!-- Task Input Form -->
    <div class="task-form">
      <input 
        v-model="newTask" 
        @keyup.enter="addTask"
        placeholder="Add a new task"
      />
      <button @click="addTask">Add Task</button>
    </div>

    <!-- Task Filters -->
    <div class="filters">
      <button 
        v-for="filter in filters" 
        :key="filter"
        :class="{ active: currentFilter === filter }"
        @click="currentFilter = filter"
      >
        {{ filter }}
      </button>
    </div>

    <!-- Task List -->
    <ul class="task-list">
      <li v-for="task in filteredTasks" :key="task.id">
        <input
          type="checkbox"
          v-model="task.completed"
        />
        <span :class="{ completed: task.completed }">
          {{ task.text }}
        </span>
        <button @click="deleteTask(task.id)" class="delete">×</button>
      </li>
    </ul>

    <!-- Task Stats -->
    <div class="stats">
      <span>{{ completedCount }} / {{ tasks.length }} completed</span>
      <button v-if="completedCount" @click="clearCompleted">
        Clear Completed
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TodoApp',
  data() {
    return {
      title: 'Vue Todo Manager',
      newTask: '',
      tasks: [],
      currentFilter: 'All',
      filters: ['All', 'Active', 'Completed']
    }
  },
  computed: {
    filteredTasks() {
      switch (this.currentFilter) {
        case 'Active':
          return this.tasks.filter(task => !task.completed)
        case 'Completed':
          return this.tasks.filter(task => task.completed)
        default:
          return this.tasks
      }
    },
    completedCount() {
      return this.tasks.filter(task => task.completed).length
    }
  },
  methods: {
    addTask() {
      if (this.newTask.trim()) {
        this.tasks.push({
          id: Date.now(),
          text: this.newTask,
          completed: false
        })
        this.newTask = ''
      }
    },
    deleteTask(id) {
      this.tasks = this.tasks.filter(task => task.id !== id)
    },
    clearCompleted() {
      this.tasks = this.tasks.filter(task => !task.completed)
    }
  },
  // Lifecycle hook - when component is mounted
  mounted() {
    // Load tasks from localStorage if available
    const savedTasks = localStorage.getItem('tasks')
    if (savedTasks) {
      this.tasks = JSON.parse(savedTasks)
    }
  },
  // Watch for changes in tasks array
  watch: {
    tasks: {
      handler(newTasks) {
        localStorage.setItem('tasks', JSON.stringify(newTasks))
      },
      deep: true
    }
  }
}
</script>

<style scoped>
.todo-app {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
  background: #f5f5f5;
  border-radius: 8px;
}

.task-form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.task-form input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #45a049;
}

.filters {
  margin-bottom: 20px;
  display: flex;
  gap: 10px;
}

.filters button {
  background: #fff;
  color: #666;
  border: 1px solid #ddd;
}

.filters button.active {
  background: #4CAF50;
  color: white;
}

.task-list {
  list-style: none;
  padding: 0;
}

.task-list li {
  display: flex;
  align-items: center;
  padding: 10px;
  background: white;
  margin-bottom: 5px;
  border-radius: 4px;
}

.completed {
  text-decoration: line-through;
  color: #999;
}

.delete {
  background: #ff4444;
  margin-left: auto;
  padding: 4px 8px;
}

.stats {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>