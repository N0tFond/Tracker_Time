<template>
  <div class="min-h-screen bg-gray-50">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto px-4 py-6 sm:px-6 lg:px-8">
        <h1 class="text-3xl font-bold text-gray-900">Suivi du Temps de Travail</h1>
      </div>
    </header>
    
    <main class="max-w-7xl mx-auto px-4 py-6 sm:px-6 lg:px-8">
      <!-- Navigation Tabs -->
      <div class="border-b border-gray-200 mb-6">
        <nav class="flex -mb-px space-x-8">
          <button 
            v-for="tab in tabs" 
            :key="tab.id"
            @click="activeTab = tab.id"
            :class="[
              activeTab === tab.id 
                ? 'border-primary-500 text-primary-600' 
                : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300',
              'whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm'
            ]"
          >
            {{ tab.name }}
          </button>
        </nav>
      </div>
      
      <!-- Time Tracking Section -->
      <div v-if="activeTab === 'timeTracking'" class="space-y-6">
        <div class="bg-white shadow rounded-lg p-6">
          <div class="flex justify-between items-center mb-6">
            <h2 class="text-xl font-semibold text-gray-800">Saisie d'Heures</h2>
            <div class="text-2xl font-mono">{{ formatTime(elapsedTime) }}</div>
          </div>
          
          <div class="flex space-x-4 mb-6">
            <button 
              @click="startTimer" 
              :disabled="isTimerRunning"
              class="px-4 py-2 bg-green-600 text-white rounded-md disabled:bg-gray-400"
            >
              Démarrer
            </button>
            <button 
              @click="stopTimer" 
              :disabled="!isTimerRunning"
              class="px-4 py-2 bg-red-600 text-white rounded-md disabled:bg-gray-400"
            >
              Arrêter
            </button>
          </div>
          
          <div class="space-y-4">
            <div>
              <label for="activity" class="block text-sm font-medium text-gray-700">Activité</label>
              <input 
                id="activity" 
                v-model="currentActivity" 
                type="text" 
                class="mt-1 pl-2 block h-10 w-full rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500"
                placeholder="Décrivez votre activité"
              />
            </div>
            
            <div>
              <label for="description" class="block text-sm font-medium text-gray-700">Description détaillée</label>
              <textarea 
                id="description" 
                v-model="currentDescription" 
                rows="3" 
                class="mt-1 pl-2 pt-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500"
                placeholder="Détails sur ce que vous avez fait"
              ></textarea>
            </div>
            
            <button 
              @click="saveTimeEntry" 
              class="w-full px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700"
            >
              Enregistrer
            </button>
          </div>
        </div>
        
        <div class="bg-white shadow rounded-lg p-6">
          <h3 class="text-lg font-medium text-gray-800 mb-4">Entrées récentes</h3>
          <div class="space-y-4">
            <div v-for="(entry, index) in timeEntries" :key="index" class="border-b pb-4">
              <div class="flex justify-between">
                <span class="font-medium">{{ entry.activity }}</span>
                <span class="text-gray-500">{{ formatTime(entry.duration) }}</span>
              </div>
              <p class="text-gray-600 text-sm mt-1">{{ entry.description }}</p>
              <div class="text-xs text-gray-400 mt-1">{{ formatDate(entry.date) }}</div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Reports Section -->
      <div v-if="activeTab === 'reports'" class="space-y-6">
        <div class="bg-white shadow rounded-lg p-6">
          <h2 class="text-xl font-semibold text-gray-800 mb-6">Rapports d'Activité</h2>
          
          <div class="mb-6">
            <h3 class="text-lg font-medium text-gray-700 mb-2">Résumé</h3>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
              <div class="bg-gray-50 p-4 rounded-lg">
                <div class="text-sm text-gray-500">Total des heures</div>
                <div class="text-2xl font-semibold">{{ formatTime(totalTime) }}</div>
              </div>
              <div class="bg-gray-50 p-4 rounded-lg">
                <div class="text-sm text-gray-500">Entrées</div>
                <div class="text-2xl font-semibold">{{ timeEntries.length }}</div>
              </div>
              <div class="bg-gray-50 p-4 rounded-lg">
                <div class="text-sm text-gray-500">Moyenne par entrée</div>
                <div class="text-2xl font-semibold">{{ formatTime(averageTime) }}</div>
              </div>
            </div>
          </div>
          
          <div>
            <h3 class="text-lg font-medium text-gray-700 mb-2">Détails par activité</h3>
            <div class="overflow-x-auto">
              <table class="min-w-full divide-y divide-gray-200">
                <thead class="bg-gray-50">
                  <tr>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Activité</th>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Temps total</th>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Entrées</th>
                  </tr>
                </thead>
                <tbody class="bg-white divide-y divide-gray-200">
                  <tr v-for="(report, activity) in activityReports" :key="activity">
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{{ activity }}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{ formatTime(report.totalTime) }}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{ report.count }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Tasks Section -->
      <div v-if="activeTab === 'tasks'" class="space-y-6">
        <div class="bg-white shadow rounded-lg p-6">
          <h2 class="text-xl font-semibold text-gray-800 mb-6">Planification de Tâches</h2>
          
          <div class="mb-6">
            <div class="flex space-x-4 mb-4">
              <input 
                v-model="newTaskTitle" 
                type="text" 
                class="flex-1 pl-2 rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500"
                placeholder="Nouvelle tâche"
              />
              <button 
                @click="addTask" 
                class="px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700"
              >
                Ajouter
              </button>
            </div>
          </div>
          
          <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
            <div v-for="column in taskColumns" :key="column.id" class="bg-gray-50 p-4 rounded-lg">
              <h3 class="font-medium text-gray-700 mb-3">{{ column.name }}</h3>
              <div class="space-y-2">
                <div 
                  v-for="task in getTasksByStatus(column.id)" 
                  :key="task.id" 
                  class="bg-white p-3 rounded shadow-sm"
                >
                  <div class="flex justify-between items-start">
                    <span class="font-medium">{{ task.title }}</span>
                    <div class="flex space-x-1">
                      <button 
                        v-if="column.id !== 'done'" 
                        @click="moveTask(task.id, getNextStatus(column.id))" 
                        class="text-xs bg-gray-200 hover:bg-gray-300 px-2 py-1 rounded"
                      >
                        →
                      </button>
                      <button 
                        @click="deleteTask(task.id)" 
                        class="text-xs bg-red-100 hover:bg-red-200 text-red-600 px-2 py-1 rounded"
                      >
                        ×
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Notes Section -->
      <div v-if="activeTab === 'notes'" class="space-y-6">
        <div class="bg-white shadow rounded-lg p-6">
          <h2 class="text-xl font-semibold text-gray-800 mb-6">Notes Rapides</h2>
          
          <div class="mb-6">
            <textarea 
              v-model="newNote" 
              rows="3" 
              class="w-full pl-2 pt-1 resize-none h-[24rem] rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500"
              placeholder="Écrivez votre note ici..."
            ></textarea>
            <div class="flex justify-end mt-2">
              <button 
                @click="addNote" 
                class="px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700"
              >
                Enregistrer
              </button>
            </div>
          </div>
          
          <div class="space-y-4">
            <div v-for="(note, index) in notes" :key="index" class="bg-yellow-50 p-4 rounded-lg relative">
              <button 
                @click="deleteNote(index)" 
                class="absolute top-2 right-2 text-gray-400 hover:text-gray-600"
              >
                ×
              </button>
              <p class="whitespace-pre-line">{{ note.content }}</p>
              <div class="text-xs text-gray-400 mt-2">{{ formatDate(note.date) }}</div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Navigation
      activeTab: 'timeTracking',
      tabs: [
        { id: 'timeTracking', name: 'Saisie d\'Heures' },
        { id: 'reports', name: 'Rapports' },
        { id: 'tasks', name: 'Tâches' },
        { id: 'notes', name: 'Notes' }
      ],
      
      // Time tracking
      elapsedTime: 0,
      timerInterval: null,
      isTimerRunning: false,
      startTime: null,
      currentActivity: '',
      currentDescription: '',
      timeEntries: [],
      
      // Tasks
      tasks: [],
      newTaskTitle: '',
      taskColumns: [
        { id: 'todo', name: 'À faire' },
        { id: 'inProgress', name: 'En cours' },
        { id: 'done', name: 'Terminé' }
      ],
      
      // Notes
      notes: [],
      newNote: ''
    }
  },
  computed: {
    totalTime() {
      return this.timeEntries.reduce((total, entry) => total + entry.duration, 0);
    },
    averageTime() {
      return this.timeEntries.length > 0 ? this.totalTime / this.timeEntries.length : 0;
    },
    activityReports() {
      const reports = {};
      
      this.timeEntries.forEach(entry => {
        if (!reports[entry.activity]) {
          reports[entry.activity] = { totalTime: 0, count: 0 };
        }
        
        reports[entry.activity].totalTime += entry.duration;
        reports[entry.activity].count += 1;
      });
      
      return reports;
    }
  },
  methods: {
    // Time tracking methods
    startTimer() {
      if (this.isTimerRunning) return;
      
      this.startTime = Date.now() - this.elapsedTime;
      this.timerInterval = setInterval(() => {
        this.elapsedTime = Date.now() - this.startTime;
      }, 1000);
      this.isTimerRunning = true;
    },
    stopTimer() {
      if (!this.isTimerRunning) return;
      
      clearInterval(this.timerInterval);
      this.isTimerRunning = false;
    },
    saveTimeEntry() {
      if (this.elapsedTime === 0 || !this.currentActivity) return;
      
      this.timeEntries.unshift({
        activity: this.currentActivity,
        description: this.currentDescription,
        duration: this.elapsedTime,
        date: new Date()
      });
      
      // Reset
      this.elapsedTime = 0;
      this.currentDescription = '';
      this.stopTimer();
    },
    formatTime(ms) {
      const seconds = Math.floor((ms / 1000) % 60);
      const minutes = Math.floor((ms / (1000 * 60)) % 60);
      const hours = Math.floor(ms / (1000 * 60 * 60));
      
      return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
    },
    formatDate(date) {
      return new Date(date).toLocaleString();
    },
    
    // Task methods
    addTask() {
      if (!this.newTaskTitle.trim()) return;
      
      this.tasks.push({
        id: Date.now().toString(),
        title: this.newTaskTitle,
        status: 'todo',
        createdAt: new Date()
      });
      
      this.newTaskTitle = '';
    },
    getTasksByStatus(status) {
      return this.tasks.filter(task => task.status === status);
    },
    moveTask(taskId, newStatus) {
      const taskIndex = this.tasks.findIndex(task => task.id === taskId);
      if (taskIndex !== -1) {
        this.tasks[taskIndex].status = newStatus;
      }
    },
    deleteTask(taskId) {
      this.tasks = this.tasks.filter(task => task.id !== taskId);
    },
    getNextStatus(currentStatus) {
      const statusOrder = ['todo', 'inProgress', 'done'];
      const currentIndex = statusOrder.indexOf(currentStatus);
      return statusOrder[currentIndex + 1] || currentStatus;
    },
    
    // Note methods
    addNote() {
      if (!this.newNote.trim()) return;
      
      this.notes.unshift({
        content: this.newNote,
        date: new Date()
      });
      
      this.newNote = '';
    },
    deleteNote(index) {
      this.notes.splice(index, 1);
    }
  }
}
</script>

<style>
:root {
  --color-primary-50: #f0f9ff;
  --color-primary-100: #e0f2fe;
  --color-primary-200: #bae6fd;
  --color-primary-300: #7dd3fc;
  --color-primary-400: #38bdf8;
  --color-primary-500: #0ea5e9;
  --color-primary-600: #0284c7;
  --color-primary-700: #0369a1;
  --color-primary-800: #075985;
  --color-primary-900: #0c4a6e;
}

.bg-primary-600 {
  background-color: var(--color-primary-600);
}

.hover\:bg-primary-700:hover {
  background-color: var(--color-primary-700);
}

.text-primary-600 {
  color: var(--color-primary-600);
}

.border-primary-500 {
  border-color: var(--color-primary-500);
}

.focus\:border-primary-500:focus {
  border-color: var(--color-primary-500);
}

.focus\:ring-primary-500:focus {
  --tw-ring-color: var(--color-primary-500);
}
</style>
