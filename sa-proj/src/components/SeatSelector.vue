<template>
  <div
    v-if="props.visible"
    class="fixed inset-0 bg-black/60 flex items-center justify-center z-50 p-4"
  >
    <div
      class="bg-white dark:bg-zinc-900 p-6 rounded-xl w-full max-w-2xl shadow-lg flex flex-col"
    >
      <h2 class="text-xl font-bold mb-4 text-center">Select Your Seats</h2>

      <!-- Screen indicator -->
      <div
        class="mx-auto mb-6 w-3/4 h-6 bg-gray-300 dark:bg-gray-700 rounded-md flex items-center justify-center text-sm font-semibold text-gray-700 dark:text-gray-300 shadow-inner"
      >
        Screen
      </div>

      <!-- Seats grid -->
      <div class="grid grid-cols-8 gap-4 justify-center mb-6">
        <div
          v-for="seat in seats"
          :key="seat.id"
          @click="toggleSeat(seat.id)"
          :class="[
            'w-12 h-12 rounded-full cursor-pointer select-none flex items-center justify-center font-semibold text-sm shadow-md transition-transform duration-200 ease-in-out',
            seat.selected
              ? 'bg-primary text-white shadow-xl scale-110'
              : 'bg-gray-100 dark:bg-gray-800 text-gray-800 dark:text-gray-300 hover:bg-primary/20 dark:hover:bg-primary/30 hover:text-primary dark:hover:text-primary scale-100'
          ]"
        >
          {{ seat.label }}
        </div>
      </div>

      <!-- Selected seats info, total price and buttons -->
      <div
        class="flex flex-col sm:flex-row justify-between items-center gap-4"
      >
        <div class="text-sm text-muted-foreground">
          Selected: {{ selectedSeats.map((s) => s.label).join(", ") || "None" }}<br />
          <span class="font-semibold">
            Total: {{ totalPrice }} THB
          </span>
        </div>
        <div class="flex gap-3">
          <button
            @click="$emit('cancel')"
            class="px-5 py-2 bg-gray-300 rounded-md hover:bg-gray-400 transition"
          >
            Back
          </button>
          <button
            @click="confirmBooking"
            :disabled="selectedSeats.length === 0"
            class="px-5 py-2 bg-primary text-white rounded-md hover:bg-primary/90 transition disabled:opacity-50"
          >
            Confirm
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, defineEmits, defineProps, watch } from "vue";

const props = defineProps<{ visible: boolean }>();
const emit = defineEmits(["confirm", "cancel"]);

const seats = ref(
  Array.from({ length: 40 }, (_, i) => ({
    id: i,
    label: `A${i + 1}`,
    selected: false,
  }))
);

watch(
  () => props.visible,
  (val) => {
    if (val) {
      seats.value.forEach((seat) => (seat.selected = false));
    }
  }
);

const selectedSeats = computed(() =>
  seats.value.filter((seat) => seat.selected)
);

const totalPrice = computed(() => selectedSeats.value.length * 120);

function toggleSeat(id: number) {
  const seat = seats.value.find((seat) => seat.id === id);
  if (seat) seat.selected = !seat.selected;
}

function confirmBooking() {
  emit("confirm", selectedSeats.value);
}
</script>
