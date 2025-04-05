<script>
export default {
    name: 'Notes',
    data() {
        return {
            notes: [],
            newNoteTitle: '',
            newNoteDescription: '',
            showDeleteModal: false,
            noteToDeleteIndex: null
        };
    },
    mounted() {
        // Charger les notes depuis le localStorage au démarrage
        this.loadFromLocalStorage();
    },
    methods: {
        addNote() {
            if (!this.newNoteTitle.trim() || !this.newNoteDescription.trim()) return;
            this.notes.unshift({
                title: this.newNoteTitle,
                content: this.newNoteDescription,
                date: new Date()
            });
            // Sauvegarder dans le localStorage
            this.saveToLocalStorage();
            this.newNoteTitle = '';
            this.newNoteDescription = '';
        },
        openDeleteModal(index) {
            this.noteToDeleteIndex = index;
            this.showDeleteModal = true;
        },
        closeDeleteModal() {
            this.showDeleteModal = false;
            this.noteToDeleteIndex = null;
        },
        confirmDelete() {
            if (this.noteToDeleteIndex !== null) {
                this.notes.splice(this.noteToDeleteIndex, 1);
                this.saveToLocalStorage();
                this.closeDeleteModal();
            }
        },
        formatDate(date) {
            return new Date(date).toLocaleString();
        },
        saveToLocalStorage() {
            // Convertir les dates en chaînes pour le stockage
            const notesToStore = this.notes.map(note => ({
                ...note,
                date: note.date.toString()
            }));

            localStorage.setItem('notes', JSON.stringify(notesToStore));
        },
        loadFromLocalStorage() {
            const storedNotes = localStorage.getItem('notes');
            if (storedNotes) {
                // Reconvertir les chaînes en objets Date
                this.notes = JSON.parse(storedNotes).map(note => ({
                    ...note,
                    date: new Date(note.date)
                }));
            }
        }
    }
};
</script>

<template>
    <div class="space-y-6">
        <div class="bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs shadow rounded-lg p-6">
            <h2 class="text-xl font-semibold text-black mb-6">Notes Rapides</h2>
            <div class="flex flex-col">
                <label for="title" class="mb-3">Entrez le titre de votre note</label>
                <input type="text" id="title" placeholder="Titre de votre note" v-model="newNoteTitle"
                    class="mb-3 rounded-md bg-white/40 border-gray-300 shadow-sm focus:outline-primary-400 pl-2 pt-1 h-8" />
            </div>
            <div class="mb-6">
                <textarea v-model="newNoteDescription" rows="3"
                    class="w-full pl-2 pt-1 resize-none h-[20rem] rounded-md bg-white/40 border-gray-300 shadow-sm focus:outline-primary-400"
                    placeholder="Écrivez votre note ici..."></textarea>
                <div class="flex justify-end mt-2">
                    <button @click="addNote"
                        class="px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700">
                        Enregistrer
                    </button>
                </div>
            </div>
            <div class="space-y-4 h-30 overflow-y-scroll overflow-x-hidden scroll-p-1 custom-scrollbar notes-container">
                <div v-for="(note, index) in notes" :key="index" class="bg-secondary/20 p-4 rounded-lg relative">
                    <button @click="openDeleteModal(index)"
                        class="absolute top-2 right-2 text-xs bg-red-100 hover:bg-red-200 text-red-600 px-2 py-1 rounded">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                        </svg>
                    </button>
                    <h3 class="text-xl font-bold mb-2">{{ note.title }}</h3>
                    <p class="whitespace-pre-line text-lg text-black/50">{{ note.content }}</p>
                    <div class="text-xs text-gray-800/50 mt-2">{{ formatDate(note.date) }}</div>
                </div>
            </div>
        </div>
    </div>
    <!-- Fenêtre modale de confirmation de suppression -->
    <div v-if="showDeleteModal"
        class="fixed inset-0 bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs flex items-center justify-center z-50">
        <div class="bg-white rounded-lg p-6 max-w-md w-full mx-4">
            <h3 class="text-lg font-medium text-gray-900 mb-4">Confirmer la suppression</h3>
            <p class="text-gray-600 mb-6">
                Êtes-vous sûr de vouloir supprimer cette note ? Cette action est irréversible.
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
    background: #f1f1f10c;
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