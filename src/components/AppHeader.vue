<template>
  <header
    class="border-b border-slate-200 bg-gradient-to-r from-sky-800 to-cyan-700 text-white shadow-sm"
  >
    <div class="max-w-6xl mx-auto px-4 py-4 flex flex-col gap-4 sm:flex-row sm:items-center">
      <div>
        <p class="text-xs uppercase tracking-wide/relaxed text-sky-100">Dashboard</p>
        <h1 class="text-2xl font-semibold tracking-tight">Learning Sessions</h1>
      </div>

      <div
        class="w-full sm:w-auto sm:ml-auto flex flex-col gap-2 sm:flex-row sm:items-center"
        role="region"
        aria-label="controls"
      >
        <div class="relative w-full sm:w-72">
          <span class="pointer-events-none absolute inset-y-0 left-3 flex items-center text-sky-300">
            🔎︎
          </span>
          <input
            id="search"
            :value="query"
            type="search"
            placeholder="Search by title..."
            class="w-full rounded-full border border-sky-200/80 bg-white/90 text-slate-800 px-9 py-2 text-sm placeholder:text-slate-400 shadow-sm focus:outline-none focus:ring-2 focus:ring-white/70"
            aria-label="Search sessions by title"
            :disabled="loading"
            @input="onQueryInput"
          />
        </div>

        <button
          @click="emit('toggle-sort')"
          class="inline-flex items-center gap-2 rounded-full border border-sky-200/80 bg-white/90 text-slate-800 font-medium px-4 py-2 text-sm shadow-sm hover:shadow-md"
          :title="sortDesc ? 'Sort ascending' : 'Sort descending'"
          :disabled="loading"
        >
          <span class="hidden sm:inline">Sort by popularity</span>
          <span aria-hidden="true">{{ sortDesc ? '↓' : '↑' }}</span>
        </button>

        <label
          class="inline-flex items-center gap-2 rounded-full border border-sky-200/80 bg-white/90 text-slate-800 font-medium px-4 py-2 text-sm shadow-sm cursor-pointer"
        >
          <input
            type="checkbox"
            :checked="simulateNextError"
            class="h-4 w-4 rounded border-sky-200/80 text-sky-500"
            :disabled="loading"
            @change="onSimErrorChange"
          />
          Simulate error
        </label>
      </div>
    </div>
  </header>
</template>

<script setup>
const props = defineProps({
  query: {
    type: String,
    default: '',
  },
  sortDesc: {
    type: Boolean,
    default: true,
  },
  simulateNextError: {
    type: Boolean,
    default: false,
  },
  loading: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['update:query', 'toggle-sort', 'update:simulateNextError'])

function onQueryInput(event) {
  emit('update:query', event.target.value)
}

function onSimErrorChange(event) {
  emit('update:simulateNextError', event.target.checked)
}
</script>


