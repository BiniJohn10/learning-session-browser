<template>
  <article
    role="listitem"
    class="flex flex-col gap-4 rounded-xl border border-slate-200/80 bg-white px-4 py-4 shadow-sm transition hover:-translate-y-0.5 hover:border-slate-300 hover:shadow-md sm:flex-row sm:items-start sm:px-5 sm:py-4"
  >
    <div class="flex-1 min-w-0">
      <div class="text-base font-semibold text-slate-900 leading-snug">
        {{ session.title }}
      </div>
      <div class="mt-2 flex flex-wrap items-center gap-2 text-sm text-slate-600">
        <span class="text-[11px] uppercase tracking-wide text-slate-500">Tags</span>
        <span
          class="inline-flex items-center rounded-full bg-cyan-50 px-2.5 py-1 text-[11px] font-medium text-cyan-900"
        >
          {{ session.tags?.length ? session.tags.join(', ') : '-' }}
        </span>

        <span class="mx-1 text-slate-300">•</span>
        <span>{{ coerceMins(session.mins) }} mins</span>

        <span class="mx-1 text-slate-300">•</span>
        <span
          class="inline-flex items-center rounded-full bg-sky-50 px-2 py-0.5 text-[11px] font-semibold uppercase tracking-wide text-sky-700"
        >
          {{ session.difficulty ? session.difficulty.trim() : 'N/A' }}
        </span>

        <span class="mx-1 text-slate-300">•</span>
        <span>Popularity: {{ session.popularity ?? 0 }}</span>
      </div>
    </div>

    <div class="flex flex-row items-end justify-between gap-3 sm:flex-col sm:items-end">
      <div class="text-xs text-slate-500">
        Updated
        <span class="font-medium text-slate-600">
          {{ formatDate(session.updatedAt) }}
        </span>
      </div>
      <button
        class="inline-flex items-center justify-center gap-2 rounded-full border px-3.5 py-1.5 text-sm font-medium shadow-sm transition focus:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:ring-indigo-500"
        :class="isCompleted(session.id)
          ? 'border-transparent bg-emerald-500 text-white hover:bg-emerald-600 focus-visible:ring-emerald-500'
          : 'border-slate-200 bg-white text-slate-800 hover:bg-slate-50'"
        :aria-pressed="isCompleted(session.id)"
        :aria-label="isCompleted(session.id) ? 'Mark incomplete' : 'Mark complete'"
        @click="toggleCompleted(session.id)"
      >
        <span
          v-if="isCompleted(session.id)"
          class="inline-flex h-4 w-4 items-center justify-center rounded-full bg-emerald-100 text-[11px] font-bold text-emerald-600"
          aria-hidden="true"
        >
          ✓
        </span>
        <span>{{ isCompleted(session.id) ? 'Completed' : 'Mark complete' }}</span>
      </button>
    </div>
  </article>
</template>

<script setup>
const props = defineProps({
  session: {
    type: Object,
    required: true,
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


