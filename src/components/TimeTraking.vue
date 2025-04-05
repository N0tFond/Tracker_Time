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
            currentImage: null,
            currentImagePreview: null,
            timeEntries: [],
            showDeleteModal: false,
            entryToDeleteIndex: null
        }
    },
    mounted() {
        // Charger les entrées depuis le localStorage au démarrage
        this.loadFromLocalStorage();
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

            const newEntry = {
                activity: this.currentActivity,
                description: this.currentDescription,
                duration: this.elapsedTime,
                date: new Date(),
                image: this.currentImage
            };

            this.timeEntries.unshift(newEntry);

            // Sauvegarder dans le localStorage
            this.saveToLocalStorage();

            // Reset
            this.elapsedTime = 0;
            this.currentActivity = '';
            this.currentDescription = '';
            this.currentImage = null;
            this.currentImagePreview = null;
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
        handleImageUpload(event) {
            const file = event.target.files[0];
            if (!file) return;

            // Vérifier si c'est une image
            if (!file.type.match('image.*')) {
                alert('Veuillez sélectionner une image');
                return;
            }

            // Créer un aperçu de l'image
            this.currentImagePreview = URL.createObjectURL(file);

            // Convertir l'image en base64 pour le stockage
            const reader = new FileReader();
            reader.onload = (e) => {
                this.currentImage = e.target.result;
            };
            reader.readAsDataURL(file);
        },
        removeImage() {
            this.currentImage = null;
            this.currentImagePreview = null;
            // Réinitialiser l'input file
            const fileInput = this.$refs.imageInput;
            if (fileInput) fileInput.value = '';
        },
        saveToLocalStorage() {
            // Convertir les dates en chaînes pour le stockage
            const entriesToStore = this.timeEntries.map(entry => ({
                ...entry,
                date: entry.date.toString()
            }));

            localStorage.setItem('timeEntries', JSON.stringify(entriesToStore));
        },
        loadFromLocalStorage() {
            const storedEntries = localStorage.getItem('timeEntries');
            if (storedEntries) {
                // Reconvertir les chaînes en objets Date
                this.timeEntries = JSON.parse(storedEntries).map(entry => ({
                    ...entry,
                    date: new Date(entry.date)
                }));
            }
        },
        openDeleteModal(index) {
            this.entryToDeleteIndex = index;
            this.showDeleteModal = true;
        },
        closeDeleteModal() {
            this.showDeleteModal = false;
            this.entryToDeleteIndex = null;
        },
        confirmDelete() {
            if (this.entryToDeleteIndex !== null) {
                this.timeEntries.splice(this.entryToDeleteIndex, 1);
                this.saveToLocalStorage();
                this.closeDeleteModal();
            }
        }
    }
}
</script>

<template>
    <div class="flex space-x-6">
        <div class="flex-1 bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs shadow rounded-lg p-6">
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
                    class="px-4 py-2 bg-red-600 text-slate-900 rounded-md disabled:bg-gray-200">
                    Arrêter
                </button>
            </div>
            <div class="space-y-4">
                <div>
                    <label for="activity" class="block text-lg font-medium text-gray-700">Activité</label>
                    <input id="activity" v-model="currentActivity" type="text"
                        class="mt-1 pl-2 block h-10 w-full rounded-md bg-white/40 border-gray-300 shadow-sm focus:outline-primary-400"
                        placeholder="Décrivez votre activité" />
                </div>
                <div>
                    <label for="description" class="block text-lg font-medium text-gray-700">Description
                        détaillée</label>
                    <textarea id="description" v-model="currentDescription" rows="3"
                        class="mt-1 pl-2 pt-2 block w-full h-[19rem] resize-none rounded-md bg-white/40 border-gray-300 shadow-sm focus:outline-primary-400"
                        placeholder="Détails sur ce que vous avez fait"></textarea>
                </div>
                <!-- Ajout de la fonctionnalité d'upload d'image -->
                <div>
                    <label for="image" class="block text-lg font-medium text-gray-700">Image</label>
                    <div class="mt-1 flex items-center">
                        <input type="file" id="image" ref="imageInput" @change="handleImageUpload" accept="image/*"
                            class="hidden" />
                        <label for="image"
                            class="px-4 py-2 bg-gray-200 text-gray-700 rounded-md cursor-pointer hover:bg-gray-300">
                            Choisir une image
                        </label>
                        <button v-if="currentImagePreview" @click="removeImage"
                            class="ml-2 px-2 py-1 bg-red-100 text-red-600 rounded-md hover:bg-red-200">
                            Supprimer
                        </button>
                    </div>
                    <!-- Aperçu de l'image -->
                    <div v-if="currentImagePreview" class="mt-2">
                        <img :src="currentImagePreview" alt="Aperçu" class="h-32 object-cover rounded-md" />
                    </div>
                </div>
                <button @click="saveTimeEntry"
                    class="w-full px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700">
                    Enregistrer
                </button>
            </div>
        </div>

        <div class="flex-1 bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs shadow rounded-lg p-6">
            <h3 class="text-xl font-bold text-black mb-4">Entrées récentes</h3>
            <div
                class="space-y-4 h-[40rem] overflow-y-scroll overflow-x-hidden scroll-p-1 custom-scrollbar notes-container">
                <div v-for="(entry, index) in timeEntries" :key="index" class="border-b pb-4">
                    <div class="flex justify-between">
                        <span class="font-bold text-xl text-primary-300">{{ entry.activity }}</span>
                        <div class="flex items-center gap-2">
                            <span class="text-gray-50">{{ formatTime(entry.duration) }}</span>
                            <button @click="openDeleteModal(index)"
                                class="text-red-500 hover:text-red-700 p-1 rounded-full hover:bg-red-100"
                                title="Supprimer cette entrée">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24"
                                    stroke="currentColor">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                        d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                                </svg>
                            </button>
                        </div>
                    </div>
                    <p class="text-gray-200 text-sm mt-1">{{ entry.description }}</p>
                    <!-- Affichage de l'image si elle existe -->
                    <div v-if="entry.image" class="mt-2">
                        <img :src="entry.image" alt="Image de l'activité" class="h-32 object-cover rounded-md" />
                    </div>
                    <div class="text-xs text-gray-50 mt-1">{{ formatDate(entry.date) }}</div>
                </div>
            </div>
        </div>
    </div>

    <!-- Fenêtre modale de confirmation de suppression -->
    <div v-if="showDeleteModal"
        class="bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs fixed inset-0 flex items-center justify-center z-50">
        <div class="bg-white rounded-lg p-6 max-w-md w-full mx-4">
            <h3 class="text-lg font-medium text-gray-900 mb-4">Confirmer la suppression</h3>
            <p class="text-gray-600 mb-6">
                Êtes-vous sûr de vouloir supprimer cette entrée ? Cette action est irréversible.
            </p>
            <div class="flex justify-end space-x-3">
                <button @click="closeDeleteModal"
                    class="px-4 py-2 bg-gray-200 text-gray-800 rounded-md hover:bg-gray-300">
                    Annuler
                </button>
                <button @click="confirmDelete" class="px-4 py-2 bg-red-600 text-white rounded-md hover:bg-red-700">
                    Supprimer
                </button>
            </div>
        </div>
    </div>
</template>


<style scoped>
/* Styles pour personnaliser la barre de défilement */
.custom-scrollbar::-webkit-scrollbar {
    width: 10px;
}

.custom-scrollbar::-webkit-scrollbar-track {
    background: #f1f1f100;
    border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
    background: #ffffff71;
    border-radius: 5px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
    background: #5555556e;
}

.notes-container {
    padding-right: 10px;
}
</style>