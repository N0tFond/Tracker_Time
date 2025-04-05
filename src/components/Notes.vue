<script>
export default {
    name: 'Notes',
    data() {
        return {
            notes: [],
            newNote: ''
        }
    },
    methods: {
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
        },
        formatDate(date) {
            return new Date(date).toLocaleString();
        }
    }
}
</script>

<template>
    <div class="space-y-6">
        <div class="bg-gray-400/20 bg-clip-padding backdrop-filter backdrop-blur-xs shadow rounded-lg p-6">
            <h2 class="text-xl font-semibold text-black mb-6">Notes Rapides</h2>

            <div class="mb-6">
                <textarea v-model="newNote" rows="3"
                    class="w-full pl-2 pt-1 resize-none h-[24rem] rounded-md border-gray-300 shadow-sm focus:outline-primary-400"
                    placeholder="Écrivez votre note ici..."></textarea>
                <div class="flex justify-end mt-2">
                    <button @click="addNote"
                        class="px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700">
                        Enregistrer
                    </button>
                </div>
            </div>

            <div class="space-y-4">
                <div v-for="(note, index) in notes" :key="index" class="bg-yellow-50/50 p-4 rounded-lg relative">
                    <button @click="deleteNote(index)" class="absolute top-2 right-2 text-gray-600 hover:text-gray-900">
                        ×
                    </button>
                    <p class="whitespace-pre-line">{{ note.content }}</p>
                    <div class="text-xs text-gray-600 mt-2">{{ formatDate(note.date) }}</div>
                </div>
            </div>
        </div>
    </div>
</template>