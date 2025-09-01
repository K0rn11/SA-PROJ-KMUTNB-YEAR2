<template>
  <div
    v-if="visible"
    class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
  >
    <div class="bg-white dark:bg-zinc-900 p-6 rounded-xl w-full max-w-md shadow-lg">
      <h2 class="text-xl font-bold mb-4">Select Showtime</h2>

      <div class="mb-6">
        <label class="block mb-2 font-semibold">Select Date</label>
        <div class="flex gap-2 flex-wrap">
          <button
            v-for="date in dates"
            :key="date"
            @click="selectedDate = date"
            :class="[
              'px-4 py-2 rounded border cursor-pointer',
              selectedDate === date
                ? 'bg-primary text-white border-primary'
                : 'bg-white text-gray-700 dark:bg-zinc-800 dark:text-gray-300 border-gray-300 dark:border-zinc-700 hover:bg-gray-100 dark:hover:bg-zinc-700'
            ]"
          >
            {{ formatDateDisplay(date) }}
          </button>
        </div>
      </div>

      <div class="mb-6">
        <label class="block mb-2 font-semibold">Select Time</label>
        <div class="flex gap-2 flex-wrap">
          <button
            v-for="time in times"
            :key="time"
            @click="selectedTime = time"
            :class="[
              'px-4 py-2 rounded border cursor-pointer',
              selectedTime === time
                ? 'bg-primary text-white border-primary'
                : 'bg-white text-gray-700 dark:bg-zinc-800 dark:text-gray-300 border-gray-300 dark:border-zinc-700 hover:bg-gray-100 dark:hover:bg-zinc-700'
            ]"
          >
            {{ time }}
          </button>
        </div>
      </div>

      <div class="flex justify-end gap-3">
        <button
          @click="onCancel"
          class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400"
        >
          Cancel
        </button>
        <button
          @click="confirmSelection"
          :disabled="!selectedDate || !selectedTime"
          class="px-4 py-2 bg-primary text-white rounded hover:bg-primary/90 disabled:opacity-50"
        >
          Next
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineEmits, defineProps } from "vue";

// Props & Emits
defineProps<{ visible: boolean }>();
const emit = defineEmits(["confirm", "cancel"]);

const selectedDate = ref("");
const selectedTime = ref("");

// สุ่มวันที่ 5 วันข้างหน้า (รูปแบบ ISO YYYY-MM-DD)
const dates = Array.from({ length: 5 }, (_, i) => {
  const d = new Date();
  d.setDate(d.getDate() + i);
  return d.toISOString().slice(0, 10);
});

// เวลาที่ให้เลือก (fixed)
const times = ["12:00", "14:30", "17:00", "19:30", "22:00"];

function formatDateDisplay(dateStr: string) {
  const d = new Date(dateStr);
  return d.toLocaleDateString(undefined, {
    weekday: "short",
    month: "short",
    day: "numeric",
  });
}

function confirmSelection() {
  emit("confirm", { date: selectedDate.value, time: selectedTime.value });
}

function onCancel() {
  emit("cancel");
}
</script>
