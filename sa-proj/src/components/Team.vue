<script setup lang="ts">
import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
  CardFooter,
} from "@/components/ui/card";
import { ref, onMounted, computed } from "vue";

import BookingModal from "@/components/BookingModal.vue";
import SeatSelector from "@/components/SeatSelector.vue";

interface MovieProps {
  id: number;
  imageUrl: string;
  title: string;
  description?: string;
  duration?: number;
  genre?: string;
}

const API_KEY = "745ee4aac9395b951f5dfbd7b2e8758c";
const IMAGE_BASE_URL = "https://image.tmdb.org/t/p/w500";

const movies = ref<MovieProps[]>([]);
const expandedMovies = ref<Set<number>>(new Set()); // เก็บ ID ของหนังที่เปิด Read more
const currentIndex = ref(0);
const itemsPerPage = 4;

const showBookingModal = ref(false);
const showSeatSelector = ref(false);
const bookingMovie = ref<MovieProps | null>(null);
const selectedDateTime = ref<{ date: string; time: string } | null>(null);

function truncate(text: string, length: number) {
  return text.length > length ? text.slice(0, length) + "..." : text;
}

async function fetchMovies() {
  try {
    const response = await fetch(
      `https://api.themoviedb.org/3/movie/now_playing?api_key=${API_KEY}&language=en-US&page=1`
    );
    const data = await response.json();
    movies.value = data.results.map((movie: any) => ({
      id: movie.id,
      title: movie.title,
      description: movie.overview,
      imageUrl: movie.poster_path ? IMAGE_BASE_URL + movie.poster_path : "",
    }));
  } catch (error) {
    console.error("Failed to fetch movies:", error);
  }
}

onMounted(() => {
  fetchMovies();
});

const visibleMovies = computed(() =>
  movies.value.slice(currentIndex.value, currentIndex.value + itemsPerPage)
);

function next() {
  if (currentIndex.value + itemsPerPage < movies.value.length) {
    currentIndex.value += itemsPerPage;
  }
}

function prev() {
  if (currentIndex.value - itemsPerPage >= 0) {
    currentIndex.value -= itemsPerPage;
  }
}

function toggleExpand(id: number) {
  if (expandedMovies.value.has(id)) {
    expandedMovies.value.delete(id);
  } else {
    expandedMovies.value.add(id);
  }
}

function startBooking(movie: MovieProps) {
  bookingMovie.value = movie;
  showBookingModal.value = true;
}

function onDateTimeConfirm(data: { date: string; time: string }) {
  selectedDateTime.value = data;
  showBookingModal.value = false;
  showSeatSelector.value = true;
}

function onSeatConfirm(seats: { id: number; label: string }[]) {
  showSeatSelector.value = false;
  alert(`Booked ${bookingMovie.value?.title} on ${selectedDateTime.value?.date} at ${selectedDateTime.value?.time}
Seats: ${seats.map((s) => s.label).join(", ")}`);
}

</script>

<template>
  <section id="movies" class="container lg:w-[75%] py-24 sm:py-32 relative">
    <div class="text-center mb-8">
      <h2 class="text-lg text-primary mb-2 tracking-wider">Now Showing</h2>
      <h2 class="text-3xl md:text-4xl font-bold">Book Your Favorite Movie</h2>
    </div>

    <div class="relative">
      <!-- Prev button -->
      <button
        @click="prev"
        :disabled="currentIndex === 0"
        class="absolute left-[-2.5rem] top-1/2 -translate-y-1/2 z-10 bg-primary text-white rounded-full w-10 h-10 flex items-center justify-center shadow-md disabled:opacity-40 transition"
      >
        ←
      </button>

      <!-- Movie Cards -->
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-8">
        <Card
          v-for="movie in visibleMovies"
          :key="movie.id"
          class="bg-muted/60 dark:bg-card flex flex-col h-full overflow-hidden group/hoverimg cursor-pointer"
        >
          <CardHeader class="p-0 gap-0">
            <div class="h-full overflow-hidden">
              <img
                :src="movie.imageUrl"
                :alt="`Poster of ${movie.title}`"
                class="w-full aspect-[2/3] object-cover saturate-0 transition-all duration-200 ease-linear group-hover/hoverimg:saturate-100 group-hover/hoverimg:scale-[1.02]"
              />
            </div>
            <CardTitle class="py-4 px-6 text-lg font-semibold truncate">
              {{ movie.title }}
            </CardTitle>
          </CardHeader>

          <CardContent class="text-muted-foreground px-6 pb-6 flex-grow text-sm">
            <span>
              {{
                expandedMovies.has(movie.id)
                  ? movie.description
                  : truncate(movie.description || "", 100)
              }}
              <span
                v-if="movie.description && movie.description.length > 100"
                class="text-primary cursor-pointer hover:underline ml-1"
                @click="toggleExpand(movie.id)"
              >
                {{ expandedMovies.has(movie.id) ? "Show less" : "Read more" }}
              </span>
            </span>
          </CardContent>

          <CardFooter class="px-6 pb-6 flex flex-col items-center gap-4">
            <button
              class="w-full bg-primary text-white py-2 rounded hover:bg-primary/90 transition"
              @click.stop="startBooking(movie)"
            >
              Book Now
            </button>
          </CardFooter>
        </Card>
      </div>

      <!-- Next button -->
      <button
        @click="next"
        :disabled="currentIndex + itemsPerPage >= movies.length"
        class="absolute right-[-2.5rem] top-1/2 -translate-y-1/2 z-10 bg-primary text-white rounded-full w-10 h-10 flex items-center justify-center shadow-md disabled:opacity-40 transition"
      >
        →
      </button>
    </div>

    <!-- Modals -->
  <BookingModal
    :visible="showBookingModal"
    @confirm="onDateTimeConfirm"
    @cancel="() => (showBookingModal = false)"
  />

  <SeatSelector
    :visible="showSeatSelector"
    @confirm="onSeatConfirm"
    @cancel="() => {
      showSeatSelector = false;
      showBookingModal = true;
    }"
/>

  </section>
</template>
