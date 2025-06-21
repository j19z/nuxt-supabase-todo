<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">To-Do List</h1>
    <div class="mb-4">
      <UInput v-model="newTask" placeholder="Enter a new task" class="mr-2" />
      <UButton label="Add Task" color="primary" @click="addTask" />
    </div>
    <div v-for="task in tasks" :key="task.id" class="mb-2 flex items-center">
      <UButton icon="i-lucide-trash" color="error" class="ml-2" @click="deleteTask(task.id)" />
      <span>{{ task.title }}</span>
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

async function deleteTask(id) {
  const {error} = await supabase.from('tasks').delete().eq('id', id);
  if (error) console.error(error);
  else await fetchTasks();
}

onMounted(fetchTasks);
</script>