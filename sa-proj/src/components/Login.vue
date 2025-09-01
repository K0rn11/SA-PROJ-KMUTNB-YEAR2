<script setup lang="ts">
import { ref } from 'vue'
import { Button } from "@/components/ui/button"
import { LogIn, X } from "lucide-vue-next"
import { auth } from '@/firebase'
import { signInWithEmailAndPassword } from "firebase/auth"
import { Alert, AlertTitle, AlertDescription } from "@/components/ui/alert"
import { AlertCircle } from "lucide-vue-next"
import { loginUser } from '@/store/store'
import { Teleport } from 'vue'

const showModal = ref(false)
const email = ref('')
const password = ref('')

const showAlert = ref(false)
const alertMessage = ref('')
const alertType = ref<'success' | 'destructive'>('destructive')

function openModal() {
  showModal.value = true
}

function closeModal() {
  showModal.value = false
  // ไม่ซ่อน alert เวลา modal ปิด เพื่อให้ alert แสดงได้
}

async function handleLogin() {
  try {
    const userCredential = await signInWithEmailAndPassword(auth, email.value, password.value)
    const user = userCredential.user
    
    loginUser(user.email ?? '')
    
    alertType.value = 'success'
    alertMessage.value = `Login successful! Welcome ${user.email}`
    showAlert.value = true

    // รอ 3 วินาที แล้วซ่อน alert และปิด modal
    setTimeout(() => {
      showAlert.value = false
      closeModal()
      email.value = ''
      password.value = ''
    }, 3000)
  } catch (error: any) {
    alertType.value = 'destructive'
    alertMessage.value = `Login failed: ${error.message}`
    showAlert.value = true
  }
}
</script>

<template>
  <!-- Login Button -->
  <Button size="sm" variant="ghost" class="w-full justify-start" @click="openModal">
    <div class="flex gap-2 items-center">
      <LogIn class="size-5" />
      <span class="block lg:hidden">Login</span>
    </div>
    <span class="sr-only">Login</span>
  </Button>

  <!-- Modal -->
  <div
    v-if="showModal"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm"
    @click.self="closeModal"
  >
    <div class="text-white bg-card shadow-md shadow-dark border w-full max-w-md p-6 rounded-xl">
      <div class="flex justify-between items-center mb-4">
        <h2 class="text-lg font-semibold">Login</h2>
        <button @click="closeModal" class="text-gray-500 hover:text-gray-800 dark:hover:text-white">
          <X class="size-5" />
        </button>
      </div>

      <!-- Login Form -->
      <form @submit.prevent="handleLogin">
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

        <div class="mb-4">
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
          <Button type="submit" size="sm">Login</Button>
        </div>
      </form>
    </div>
  </div>

  <!-- Alert (Teleport to body so it appears fixed and outside modal) -->
  <Teleport to="body">
    <Alert
      v-if="showAlert"
      :variant="alertType"
      class="fixed bottom-6 right-6 z-50 w-full max-w-sm shadow-lg"
      :key="alertMessage"
    >
      <AlertCircle class="w-4 h-4" />
      <AlertTitle>{{ alertType === 'success' ? 'Success' : 'Error' }}</AlertTitle>
      <AlertDescription>{{ alertMessage }}</AlertDescription>
    </Alert>
  </Teleport>
</template>
