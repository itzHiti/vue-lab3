<template>
  <div class="task-container">
    <header class="task-header">
      <h2>Genius To-Do List</h2>
    </header>

    <div class="task-inputs">
      <input 
        v-model="newTaskTitle" 
        class="task-input" 
        placeholder="Enter a task..." 
        @keyup.enter="addTask" 
      />
      <select v-model="newTaskPriority" class="task-priority">
        <option value="low">Low</option>
        <option value="medium">Medium</option>
        <option value="high">High</option>
      </select>
      <button class="add-task-btn" @click="addTask">Add</button>
    </div>

    <transition-group name="fade" tag="ul" class="task-list">
      <li 
        v-for="task in sortedTasks" 
        :key="task.id" 
        :class="['task-item', taskClass(task)]"
      >
        <span 
          class="task-title" 
          @click="toggleTaskCompletion(task)" 
          :class="{ completed: task.completed }"
        >
          {{ task.title }}
        </span>
        <button class="delete-task-btn" @click="deleteTask(task.id)">
          &times;
        </button>
      </li>
    </transition-group>

    <footer class="task-footer">
      <p><strong>Pending tasks:</strong> {{ pendingTasks }}</p>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, nextTick, Ref } from 'vue';
import { Task } from '@/assets/Task';

const tasks = ref([] as Task[]);
const newTaskTitle = ref('');
const newTaskPriority = ref<'low' | 'medium' | 'high'>('low');
let taskId = 1;

const addTask = async () => {
  if (newTaskTitle.value.trim() === '' || tasks.value.some((task) => task.title === newTaskTitle.value)) return;
  const newTask: Task = {
    id: taskId++,
    title: newTaskTitle.value,
    priority: newTaskPriority.value,
    completed: false,
  };
  tasks.value.push(newTask);
  newTaskTitle.value = '';

  await nextTick();
  document.querySelector('.task-list')?.scrollIntoView({ behavior: 'smooth', block: 'end' });
};

const toggleTaskCompletion = (task: Task) => {
  if (!task) return;
  task.completed = !task.completed;
};


const deleteTask = (id: number) => {
  tasks.value = tasks.value.filter((task) => task.id !== id);
};

const pendingTasks = computed(() => tasks.value.filter((task) => !task.completed).length);

const sortedTasks = computed(() =>
  [...tasks.value].sort((a, b) => {
    const priorityOrder = { low: 1, medium: 2, high: 3 };
    return priorityOrder[b.priority] - priorityOrder[a.priority];
  })
);

const taskClass = (task: Task) => ({
  'task-low': task.priority === 'low',
  'task-medium': task.priority === 'medium',
  'task-high': task.priority === 'high',
  'task-completed': task.completed,
});

watch(
  () => tasks.value,
  (newTasks, oldTasks) => {
    console.log('Task list changed:', { newTasks, oldTasks });
  },
  { deep: true }
);
</script>

<style scoped>
.task-container {
  max-width: 400px;
  margin: 40px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  background: #f9f9f9;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.task-header {
  text-align: center;
  margin-bottom: 20px;
}

.task-inputs {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.task-input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.task-priority {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.add-task-btn {
  padding: 8px 16px;
  border: none;
  background-color: #007bff;
  color: #fff;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.add-task-btn:hover {
  background-color: #0056b3;
}

.task-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.task-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background: #fff;
  transition: transform 0.3s;
}

.task-item:hover {
  transform: scale(1.02);
}

.task-title {
  flex: 1;
  cursor: pointer;
}

.task-title.completed {
  text-decoration: line-through;
  color: #aaa;
}

.task-priority-label {
  margin-left: 10px;
  font-size: 0.9rem;
  color: #666;
}

.delete-task-btn {
  padding: 4px 8px;
  border: none;
  background: #ff4d4f;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

.delete-task-btn:hover {
  background: #d9363e;
}

.task-high {
  border-left: 5px solid red;
}

.task-medium {
  border-left: 5px solid orange;
}

.task-low {
  border-left: 5px solid lime;
}

.task-completed {
  opacity: 0.6;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
</style>
