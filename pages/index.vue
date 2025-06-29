<template>
  <div class="min-h-screen flex items-center justify-center">
    <div class="max-w-md w-full mx-auto p-4">
      <UCard class="p-6 bg-blend-darken shadow-md rounded-lg">
        <template #header>
          <h1 class="text-2xl font-bold mb-1 text-center">To-Do List</h1>
        </template>
        <div class="mb-2">
          <UInput v-model="newTask" placeholder="Enter a new task" class="mr-2" />
          <UButton label="Add Task" color="primary" @click="addTask" />
        </div>
        <div class="space-y-4">
          <div v-for="task in tasks" :key="task.id" class="flex items-center p-0.5 border-b border-gray-200">
            <UCheckbox v-model="task.completed" @change="updateTask(task)" class="mr-2" />
            <span v-if="!isEditing[task.id]" :class="{ 'line-through': task.completed }">{{ task.title }}</span>
            <UInput v-else v-model="editedTask" class="mr-2 flex-1" />
            <UButton v-if="!isEditing[task.id]" label="Edit" size="xs" color="secondary" class="ml-2" @click="startEdit(task)" />
            <UButton v-else label="Save" size="xs" color="success" class="ml-2" @click="saveEdit(task)" />
            <UButton label="Delete" size="xs" color="error" class="ml-2" @click="deleteTask(task.id)" />
          </div>
        </div>
      </UCard>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
const supabase = useSupabaseClient();
const newTask = ref('');
const tasks = ref([]);
const isEditing = ref({});
const editedTask = ref('');

async function addTask() {
  if (newTask.value.trim()) {
    const { error } = await supabase.from('tasks').insert({
      title: newTask.value,
      completed: false
    });
    if (error) console.error(error);
    else newTask.value = '';
    await fetchTasks();
  }
}

async function updateTask(task) {
  const { error } = await supabase.from('tasks').update({ completed: task.completed }).eq('id', task.id);
  if (error) console.error(error);
  else await fetchTasks();
}

async function deleteTask(id) {
  const { error } = await supabase.from('tasks').delete().eq('id', id);
  if (error) console.error(error);
  else await fetchTasks();
}

function startEdit(task) {
  isEditing.value[task.id] = true;
  editedTask.value = task.title;
}

async function saveEdit(task) {
  if (editedTask.value.trim()) {
    const { error } = await supabase.from('tasks').update({ title: editedTask.value }).eq('id', task.id);
    if (error) console.error(error);
    else {
      isEditing.value[task.id] = false;
      await fetchTasks();
    }
  }
}

async function fetchTasks() {
  const { data, error } = await supabase.from('tasks').select('*');
  if (error) console.error(error);
  else tasks.value = data;
}

onMounted(fetchTasks);
</script>