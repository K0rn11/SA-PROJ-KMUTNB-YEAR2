<script setup lang="ts">
import { ref } from 'vue'
import { Button } from "@/components/ui/button"
import { LogIn, X } from "lucide-vue-next"
import { auth } from '@/firebase'
import { createUserWithEmailAndPassword } from "firebase/auth"
import { Alert, AlertTitle, AlertDescription } from "@/components/ui/alert"
import { AlertCircle } from "lucide-vue-next"
import { loginUser } from '@/store/store'

createUserWithEmailAndPassword
const showModal = ref(false)

function openModal() {
  showModal.value = true
}

function closeModal() {
  showModal.value = false
}

// Inputs
const username = ref('')
const phone = ref('')
const email = ref('')
const password = ref('')

// Alert state
const showAlert = ref(false)
const alertMessage = ref('')
const alertType = ref<'success' | 'destructive'>('destructive')

async function handleSignup() {
  try {
    await createUserWithEmailAndPassword(auth, email.value, password.value)

    loginUser(email.value, username.value)  

    alertType.value = 'success'
    alertMessage.value = 'Signup successful! You can now login.'
    showAlert.value = true

    setTimeout(() => {
      showAlert.value = false
      closeModal()
      username.value = ''
      phone.value = ''
      email.value = ''
      password.value = ''
    }, 3000)
  } catch (error: any) {
    alertType.value = 'destructive'
    alertMessage.value = `Signup failed: ${error.message}`
    showAlert.value = true
  }
}
</script>

<template>
  <!-- Sign Up Button -->
  <Button size="sm" variant="ghost" class="w-full justify-start" @click="openModal">
    <div class="flex gap-2 items-center">
      <LogIn class="size-5" />
      <span class="block lg:hidden">Sign Up</span>
    </div>
    <span class="sr-only">Sign Up</span>
  </Button>

  <!-- Modal -->
  <div
    v-if="showModal"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm"
    @click.self="closeModal"
  >
    <div class="text-white bg-card shadow-md shadow-dark border w-full max-w-md p-6 rounded-xl">
      <div class="flex justify-between items-center mb-4">
        <h2 class="text-lg font-semibold">Sign Up</h2>
        <button @click="closeModal" class="text-gray-500 hover:text-gray-800 dark:hover:text-white">
          <X class="size-5" />
        </button>
      </div>

      <!-- Alert -->
      <Alert v-if="showAlert" :variant="alertType" class="mb-4">
        <AlertCircle class="w-4 h-4" />
        <AlertTitle>{{ alertType === 'success' ? 'Success' : 'Error' }}</AlertTitle>
        <AlertDescription>{{ alertMessage }}</AlertDescription>
      </Alert>

      <!-- Sign Up Form -->
      <form @submit.prevent="handleSignup">
        <div class="mb-4">
          <label class="block text-sm font-medium mb-1">Username</label>
          <input
            v-model="username"
            type="text"
            class="text-black w-full border rounded px-3 py-2 text-sm"
            placeholder="Username"
            required
          />
        </div>

        <div class="mb-4">
          <label class="block text-sm font-medium mb-1">Phone Number</label>
          <input
            v-model="phone"
            type="tel"
            class="text-black w-full border rounded px-3 py-2 text-sm"
            placeholder="Phone Number"
            required
          />
        </div>

        <div class="mb-4">
          <label class="block text-sm font-medium mb-1">Email</label>
          <input
            v-model="email"
            type="email"
            class="text-black w-full border rounded px-3 py-2 text-sm"
            placeholder="you@example.com"
            required
          />
        </div>

        <div class="mb-6">
          <label class="block text-sm font-medium mb-1">Password</label>
          <input
            v-model="password"
            type="password"
            class="text-black w-full border rounded px-3 py-2 text-sm"
            placeholder="Password"
            required
          />
        </div>

        <div class="flex justify-end">
          <Button type="submit" size="sm">Create Account</Button>
        </div>
      </form>
    </div>
  </div>
</template>
