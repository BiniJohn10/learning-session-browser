<template>
  <main
    role="list"
    aria-label="Learning sessions list"
    class="space-y-3"
  >
    <template v-if="sessions.length">
      <SessionCard
        v-for="s in sessions"
        :key="s.id"
        :session="s"
        :coerce-mins="coerceMins"
        :format-date="formatDate"
        :is-completed="isCompleted"
        :toggle-completed="toggleCompleted"
      />
    </template>

    <div
      v-else
      class="flex flex-col items-center justify-center gap-3 rounded-xl border border-dashed border-slate-200 bg-slate-50/80 px-6 py-10 text-center text-slate-500"
    >
      <div class="text-sm font-medium">No sessions match your search.</div>
      <p class="max-w-xs text-xs text-slate-400">
        Try a different keyword or clear the search box to see all sessions again.
      </p>
    </div>
  </main>
</template>

<script setup>
import SessionCard from './SessionCard.vue'

const props = defineProps({
  sessions: {
    type: Array,
    default: () => [],
  },
  coerceMins: {
    type: Function,
    required: true,
  },
  formatDate: {
    type: Function,
    required: true,
  },
  isCompleted: {
    type: Function,
    required: true,
  },
  toggleCompleted: {
    type: Function,
    required: true,
  },
})
</script>


