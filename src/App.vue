<script>
import TimeTracking from '@/components/TimeTraking.vue';
import Reports from '@/components/Reports.vue';
import Tasks from '@/components/Task.vue';
import Notes from '@/components/Notes.vue';

export default {
  name: 'App',
  components: {
    TimeTracking,
    Reports,
    Tasks,
    Notes
  },
  data() {
    return {
      activeTab: 'timeTracking',
      tabs: [
        { id: 'timeTracking', name: 'Saisie d\'Heures' },
        { id: 'reports', name: 'Rapports' },
        { id: 'tasks', name: 'Tâches' },
        { id: 'notes', name: 'Notes' }
      ]
    }
  }
}
</script>

<template>
  <div class="min-h-screen bg-[url('@/assets/img/backk.jpg')] bg-cover bg-center">
    <header>
      <div class="max-w-7xl mx-auto px-4 py-6 sm:px-6 lg:px-8">
        <h1 class="text-3xl font-bold text-white">Suivi du Temps de Travail</h1>
      </div>
    </header>

    <main class="max-w-7xl mx-auto px-4 py-6 sm:px-6 lg:px-8">
      <!-- Navigation Tabs -->
      <div class="border-b border-gray-200 mb-6">
        <nav class="flex -mb-px space-x-8">
          <button v-for="tab in tabs" :key="tab.id" @click="activeTab = tab.id" :class="[
            activeTab === tab.id
              ? 'border-primary-400 text-primary-500'
              : 'border-transparent text-white hover:text-gray-300 hover:border-gray-300',
            'whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm'
          ]">
            {{ tab.name }}
          </button>
        </nav>
      </div>

      <!-- Dynamic Components -->
      <TimeTracking v-if="activeTab === 'timeTracking'" />
      <Reports v-if="activeTab === 'reports'" :timeEntries="$refs.timeTracking?.timeEntries || []" />
      <Tasks v-if="activeTab === 'tasks'" />
      <Notes v-if="activeTab === 'notes'" />
    </main>
  </div>
</template>