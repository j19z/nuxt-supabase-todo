<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">To-Do List</h1>
    <div class="mb-4">
      <UInput v-model="newTask" placeholder="Enter a new task" class="mr-2" />
      <UButton label="Add Task" color="primary" @click="addTask" />
    </div>
    <div v-for="task in tasks" :key="task.id" class="mb-2">
      {{ task.title }}
    </div>
  </div>
</template>
<script setup>
import { ref, onMounted } from 'vue';
const supabase = useSupabaseClient();
const newTask = ref('');
const tasks = ref([]);

async function addTask() {
  if (newTask.value.trim()) {
    const { error } = await supabase.from('tasks').insert({
      title: newTask.value,
      completed: false
    });
    if (error) console.error(error);
    else newTask.value = '';
    await fetchTasks(); // Refresh task list
  }
}

async function fetchTasks() {
  const { data, error } = await supabase.from('tasks').select('*');
  if (error) console.error(error);
  else tasks.value = data;
}

onMounted(fetchTasks);
</script>