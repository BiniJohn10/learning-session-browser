<template>
  <div class="min-h-screen bg-slate-100 text-slate-900" aria-live="polite">
    <header class="border-b border-slate-200 bg-gradient-to-r from-sky-800 to-cyan-700 text-white shadow-sm">
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
              v-model="query"
              type="search"
              placeholder="Search by title..."
              class="w-full rounded-full border border-sky-200/80 bg-white/90 text-slate-800 px-9 py-2 text-sm placeholder:text-slate-400 shadow-sm focus:outline-none focus:ring-2 focus:ring-white/70"
              aria-label="Search sessions by title"
              :disabled="loading"
            />
          </div>

          <button
            @click="toggleSort"
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
              v-model="simulateNextError"
              class="h-4 w-4 rounded border-sky-200/80 text-sky-500"
              :disabled="loading"
            />
            Simulate error
          </label>
        </div>
      </div>
    </header>

    <section
      :aria-busy="loading ? 'true' : 'false'"
      class="max-w-6xl mx-auto px-4 py-8 space-y-4"
    >
      <div
        v-if="error"
        role="alert"
        class="relative overflow-hidden rounded-xl border border-red-200/70 bg-red-50/90 px-4 py-4 sm:px-6 sm:py-5 shadow-sm"
      >
        <div class="absolute inset-y-0 left-0 w-1 bg-gradient-to-b from-red-400 to-red-500"></div>
        <div class="ml-4 sm:ml-5">
          <strong class="text-sm font-semibold text-red-800 flex items-center gap-2">
            <span class="inline-flex h-6 w-6 items-center justify-center rounded-full bg-red-100 text-red-600 text-sm">
              !
            </span>
            Something went wrong.
          </strong>
          <div class="mt-1 text-sm text-red-700">
            We couldn't load sessions. Please check your connection and try again.
          </div>
          <div class="mt-3">
            <button
              @click="retry"
              :disabled="loading"
              class="inline-flex items-center gap-2 rounded-full bg-red-600 px-4 py-2 text-sm font-medium text-white shadow-sm transition hover:bg-red-700 hover:shadow-md"
            >
              <span>Retry</span>
              <span aria-hidden="true">⟳</span>
            </button>
          </div>
        </div>
      </div>

      <div
        v-else-if="loading"
        role="status"
        class="flex items-center gap-3 rounded-xl border border-slate-200/80 bg-white px-4 py-3 text-sm font-medium text-slate-600 shadow-sm"
      >
        <span class="inline-flex h-4 w-4 animate-spin rounded-full border-2 border-slate-300 border-t-slate-500"></span>
        Loading sessions…
      </div>

      <main
        v-else
        role="list"
        aria-label="Learning sessions list"
        class="space-y-3"
      >
        <template v-if="displayedSessions.length">
          <article
            v-for="s in displayedSessions"
            :key="s.id"
            role="listitem"
            class="flex flex-col gap-4 rounded-xl border border-slate-200/80 bg-white px-4 py-4 shadow-sm transition hover:-translate-y-0.5 hover:border-slate-300 hover:shadow-md sm:flex-row sm:items-start sm:px-5 sm:py-4"
          >
            <div class="flex-1 min-w-0">
              <div class="text-base font-semibold text-slate-900 leading-snug">
                {{ s.title }}
              </div>
              <div class="mt-2 flex flex-wrap items-center gap-2 text-sm text-slate-600">
                <span class="text-[11px] uppercase tracking-wide text-slate-500">Tags</span>
                <span
                  class="inline-flex items-center rounded-full bg-cyan-50 px-2.5 py-1 text-[11px] font-medium text-cyan-900"
                >
                  {{ s.tags?.length ? s.tags.join(', ') : '-' }}
                </span>

                <span class="mx-1 text-slate-300">•</span>
                <span>{{ coerceMins(s.mins) }} mins</span>

                <span class="mx-1 text-slate-300">•</span>
                <span
                  class="inline-flex items-center rounded-full bg-sky-50 px-2 py-0.5 text-[11px] font-semibold uppercase tracking-wide text-sky-700"
                >
                  {{ s.difficulty ? s.difficulty.trim() : 'N/A' }}
                </span>

                <span class="mx-1 text-slate-300">•</span>
                <span>Popularity: {{ s.popularity ?? 0 }}</span>
              </div>
            </div>

            <div class="flex flex-row items-end justify-between gap-3 sm:flex-col sm:items-end">
              <div class="text-xs text-slate-500">
                Updated
                <span class="font-medium text-slate-600">
                  {{ formatDate(s.updatedAt) }}
                </span>
              </div>
              <button
                class="inline-flex items-center justify-center gap-2 rounded-full border px-3.5 py-1.5 text-sm font-medium shadow-sm transition focus:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:ring-indigo-500"
                :class="isCompleted(s.id)
                  ? 'border-transparent bg-emerald-500 text-white hover:bg-emerald-600 focus-visible:ring-emerald-500'
                  : 'border-slate-200 bg-white text-slate-800 hover:bg-slate-50'"
                :aria-pressed="isCompleted(s.id)"
                :aria-label="isCompleted(s.id) ? 'Mark incomplete' : 'Mark complete'"
                @click="toggleCompleted(s.id)"
              >
                <span
                  v-if="isCompleted(s.id)"
                  class="inline-flex h-4 w-4 items-center justify-center rounded-full bg-emerald-100 text-[11px] font-bold text-emerald-600"
                  aria-hidden="true"
                >
                  ✓
                </span>
                <span>{{ isCompleted(s.id) ? 'Completed' : 'Mark complete' }}</span>
              </button>
            </div>
          </article>
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
    </section>
  </div>
</template>

<script setup>
import { ref, reactive, watch, onMounted, computed } from 'vue'

const SIM_ERROR_KEY = 'next-error'

function readSimError() {
  try { return localStorage.getItem(SIM_ERROR_KEY) === '1' } catch { return false }
}
function writeSimError(v) {
  try { localStorage.setItem(SIM_ERROR_KEY, v ? '1' : '0') } catch {}
}

const loading = ref(false)
const error = ref(null)
const sessions = ref([])
const query = ref('')
const debouncedQuery = ref('')
const sortDesc = ref(true)
const simulateNextError = ref(readSimError())
const completedIds = reactive(new Set())

const initialLoad = ref(true)

let debounceTimer = null

function coerceMins(mins) {
  const n = Number(mins)
  return Number.isFinite(n) ? n : 0
}

function formatDate(dt) {
  try { return new Date(dt).toLocaleDateString() } catch { return '—' }
}

function escapeRegex(s) {
  return s.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
}

function sortByPopularity(list, desc = true) {
  return [...list].sort((a, b) => {
    const diff = (a.popularity ?? 0) - (b.popularity ?? 0)
    if (diff === 0) return a._originalIndex - b._originalIndex
    return desc ? -diff : diff
  })
}

const displayedSessions = computed(() => {
  let list = sessions.value
  const q = debouncedQuery.value.trim()

  if (q) {
    const re = new RegExp(escapeRegex(q), 'i')
    list = list.filter(s => re.test(s.title || ''))
  }

  return sortByPopularity(list, sortDesc.value)
})

async function fetchSessionsSimulated() {
  loading.value = true
  error.value = null

  try {
    if (initialLoad.value) {
      await new Promise(r => setTimeout(r, 500))
      initialLoad.value = false
    }

    if (simulateNextError.value) {
      simulateNextError.value = false
      writeSimError(false)
      throw new Error('Simulated network error')
    }

    const res = await fetch('/sessions.json')
    if (!res.ok) throw new Error('HTTP ' + res.status)

    const data = await res.json()

    const cloned = data.map((s, i) => ({ ...s, _originalIndex: i }))
    sessions.value = cloned

    completedIds.clear()
    cloned.forEach(s => {
      if (s.completed) completedIds.add(s.id)
    })
  } catch (err) {
    error.value = err?.message ?? 'Error'
    sessions.value = []
  } finally {
    loading.value = false
  }
}

watch(query, () => {
  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(() => {
    debouncedQuery.value = query.value
  }, 300)
})

watch(simulateNextError, v => writeSimError(v))

function toggleSort() {
  sortDesc.value = !sortDesc.value
}

function isCompleted(id) {
  return completedIds.has(id)
}

function toggleCompleted(id) {
  isCompleted(id) ? completedIds.delete(id) : completedIds.add(id)
}

function retry() {
  if (!loading.value) fetchSessionsSimulated()
}

onMounted(() => {
  fetchSessionsSimulated()
})
</script>
