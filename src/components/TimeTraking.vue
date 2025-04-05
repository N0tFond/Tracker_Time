<script>
export default {
    name: 'TimeTracking',
    data() {
        return {
            elapsedTime: 0,
            timerInterval: null,
            isTimerRunning: false,
            startTime: null,
            currentActivity: '',
            currentDescription: '',
            timeEntries: []
        }
    },
    methods: {
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
        }
    }
}
</script>

<template>
    <div class="space-y-6">
        <div class="bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs shadow rounded-lg p-6">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-semibold text-gray-800">Saisie d'Heures</h2>
                <div class="text-2xl font-mono">{{ formatTime(elapsedTime) }}</div>
            </div>

            <div class="flex space-x-4 mb-6">
                <button @click="startTimer" :disabled="isTimerRunning"
                    class="px-4 py-2 bg-green-600 text-white rounded-md disabled:bg-gray-400">
                    Démarrer
                </button>
                <button @click="stopTimer" :disabled="!isTimerRunning"
                    class="px-4 py-2 bg-red-600 text-slate-500 rounded-md disabled:bg-gray-200">
                    Arrêter
                </button>
            </div>

            <div class="space-y-4">
                <div>
                    <label for="activity" class="block text-sm font-medium text-gray-700">Activité</label>
                    <input id="activity" v-model="currentActivity" type="text"
                        class="mt-1 pl-2 block h-10 w-full rounded-md border-gray-300 shadow-sm focus:outline-primary-400"
                        placeholder="Décrivez votre activité" />
                </div>

                <div>
                    <label for="description" class="block text-sm font-medium text-gray-700">Description
                        détaillée</label>
                    <textarea id="description" v-model="currentDescription" rows="3"
                        class="mt-1 pl-2 pt-2 block w-full rounded-md border-gray-300 shadow-sm focus:outline-primary-400"
                        placeholder="Détails sur ce que vous avez fait"></textarea>
                </div>

                <button @click="saveTimeEntry"
                    class="w-full px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700">
                    Enregistrer
                </button>
            </div>
        </div>

        <div class="bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs shadow rounded-lg p-6">
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
</template>