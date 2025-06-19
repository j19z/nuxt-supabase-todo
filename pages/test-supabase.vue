<template>
  <div>
    <h1>Supabase Test</h1>
    <p>{{ tasks }}</p>
  </div>
</template>

<script setup>
const supabase = useSupabaseClient();
const tasks = ref([]);

async function fetchTasks() {
  const { data, error } = await supabase.from('tasks').select('*');
  if (error) console.error(error);
  else tasks.value = data;
}

onMounted(fetchTasks);
</script>