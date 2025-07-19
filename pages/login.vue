<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-50">
    <div class="max-w-md w-full mx-auto p-4">
      <UCard class="p-6 shadow-lg">
        <template #header>
          <h1 class="text-2xl font-bold text-center">Login</h1>
          <p class="text-gray-600 text-center mt-2">Sign in to your account</p>
        </template>
        
        <form @submit.prevent="handleLogin" class="space-y-4">
          <div>
            <UFormGroup label="Email" required>
              <UInput 
                v-model="email" 
                type="email" 
                placeholder="Enter your email"
                :error="errors.email"
                required
              />
            </UFormGroup>
          </div>
          
          <div>
            <UFormGroup label="Password" required>
              <UInput 
                v-model="password" 
                type="password" 
                placeholder="Enter your password"
                :error="errors.password"
                required
              />
            </UFormGroup>
          </div>
          
          <UAlert 
            v-if="errorMessage" 
            color="red" 
            variant="soft" 
            :title="errorMessage"
            class="mb-4"
          />
          
          <UButton 
            type="submit" 
            color="primary" 
            block 
            :loading="loading"
            :disabled="loading"
          >
            {{ loading ? 'Signing in...' : 'Sign In' }}
          </UButton>
        </form>
        
        <template #footer>
          <div class="text-center">
            <p class="text-sm text-gray-600">
              Don't have an account? 
              <NuxtLink to="/signup" class="text-primary-500 hover:text-primary-600">
                Sign up
              </NuxtLink>
            </p>
          </div>
        </template>
      </UCard>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const supabase = useSupabaseClient()
const user = useSupabaseUser()

const email = ref('')
const password = ref('')
const loading = ref(false)
const errorMessage = ref('')
const errors = ref({})

// Redirect if already logged in
watch(user, (newUser) => {
  if (newUser) {
    navigateTo('/')
  }
}, { immediate: true })

async function handleLogin() {
  loading.value = true
  errorMessage.value = ''
  errors.value = {}
  
  // Basic validation
  if (!email.value) {
    errors.value.email = 'Email is required'
    loading.value = false
    return
  }
  
  if (!password.value) {
    errors.value.password = 'Password is required'
    loading.value = false
    return
  }
  
  try {
    const { error } = await supabase.auth.signInWithPassword({
      email: email.value,
      password: password.value
    })
    
    if (error) {
      errorMessage.value = error.message
    } else {
      // Success - redirect will happen via watcher
      await navigateTo('/')
    }
  } catch (err) {
    errorMessage.value = 'An unexpected error occurred'
    console.error(err)
  } finally {
    loading.value = false
  }
}

// Set page meta
definePageMeta({
  layout: false
})
</script>