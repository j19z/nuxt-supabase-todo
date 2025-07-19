<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-50">
    <div class="max-w-md w-full mx-auto p-4">
      <UCard class="p-6 shadow-lg">
        <template #header>
          <h1 class="text-2xl font-bold text-center">Sign Up</h1>
          <p class="text-gray-600 text-center mt-2">Create your account</p>
        </template>
        
        <form @submit.prevent="handleSignup" class="space-y-4">
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
          
          <div>
            <UFormGroup label="Confirm Password" required>
              <UInput 
                v-model="confirmPassword" 
                type="password" 
                placeholder="Confirm your password"
                :error="errors.confirmPassword"
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
          
          <UAlert 
            v-if="successMessage" 
            color="green" 
            variant="soft" 
            :title="successMessage"
            class="mb-4"
          />
          
          <UButton 
            type="submit" 
            color="primary" 
            block 
            :loading="loading"
            :disabled="loading"
          >
            {{ loading ? 'Creating account...' : 'Sign Up' }}
          </UButton>
        </form>
        
        <template #footer>
          <div class="text-center">
            <p class="text-sm text-gray-600">
              Already have an account? 
              <NuxtLink to="/login" class="text-primary-500 hover:text-primary-600">
                Sign in
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
const confirmPassword = ref('')
const loading = ref(false)
const errorMessage = ref('')
const successMessage = ref('')
const errors = ref({})

// Redirect if already logged in
watch(user, (newUser) => {
  if (newUser) {
    navigateTo('/')
  }
}, { immediate: true })

async function handleSignup() {
  loading.value = true
  errorMessage.value = ''
  successMessage.value = ''
  errors.value = {}
  
  // Validation
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
  
  if (password.value.length < 6) {
    errors.value.password = 'Password must be at least 6 characters'
    loading.value = false
    return
  }
  
  if (password.value !== confirmPassword.value) {
    errors.value.confirmPassword = 'Passwords do not match'
    loading.value = false
    return
  }
  
  try {
    const { error } = await supabase.auth.signUp({
      email: email.value,
      password: password.value
    })
    
    if (error) {
      errorMessage.value = error.message
    } else {
      successMessage.value = 'Account created successfully! Please check your email to verify your account.'
      // Clear form
      email.value = ''
      password.value = ''
      confirmPassword.value = ''
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