<template>
  <div :class="['task-item', { completed: task.completed }]">
    <div class="task-header">
      <div class="task-checkbox">
        <input
          type="checkbox"
          :checked="task.completed"
          @change="handleCheckboxChange"
        />
      </div>
      <div class="task-title">
        <h4>{{ task.title }}</h4>
      </div>
      <div class="task-actions">
        <button @click="toggleDetails" class="details-btn">
          {{ showDetails ? 'Hide' : 'Show' }} Details
        </button>
        <button @click="$emit('delete', task.id)" class="delete-btn">
          Delete
        </button>
      </div>
    </div>

    <div v-if="showDetails" class="task-details">
      <p class="description">{{ task.description }}</p>
      <div class="metadata">
        <span class="priority" :class="task.priority">
          Priority: {{ task.priority }}
        </span>
        <span class="category">
          Category: {{ task.category }}
        </span>
        <span class="due-date">
          Due: {{ formatDate(task.dueDate) }}
        </span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TaskItem',
  props: {
    task: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      showDetails: false
    }
  },
  methods: {
    toggleDetails() {
      this.showDetails = !this.showDetails
    },
    handleCheckboxChange() {
      this.$emit('update', {
        ...this.task,
        completed: !this.task.completed
      })
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString()
    }
  }
}
</script>

<style scoped>
.task-item {
  background: white;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.task-header {
  display: flex;
  align-items: center;
  gap: 10px;
}

.task-title {
  flex: 1;
}

.task-actions {
  display: flex;
  gap: 10px;
}

.details-btn {
  background: #2196F3;
}

.delete-btn {
  background: #ff4444;
}

.task-details {
  margin-top: 10px;
  padding-top: 10px;
  border-top: 1px solid #eee;
}

.metadata {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
}

.priority.high {
  color: #ff4444;
}

.priority.medium {
  color: #ff9800;
}

.priority.low {
  color: #4CAF50;
}

.completed {
  opacity: 0.7;
}

.completed .task-title {
  text-decoration: line-through;
}
</style>