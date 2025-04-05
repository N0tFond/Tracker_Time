<script>
export default {
    name: 'Tasks',
    data() {
        return {
            tasks: [],
            newTaskTitle: '',
            taskColumns: [
                { id: 'todo', name: 'À faire' },
                { id: 'inProgress', name: 'En cours' },
                { id: 'done', name: 'Terminé' }
            ],
            showDeleteModal: false,
            taskToDeleteId: null,
            draggedTaskId: null
        }
    },
    mounted() {
        // Charger les entrées depuis le localStorage au démarrage
        this.loadFromLocalStorage();
    },
    methods: {
        addTask() {
            if (!this.newTaskTitle.trim()) return;

            this.tasks.push({
                id: Date.now().toString(),
                title: this.newTaskTitle,
                status: 'todo',
                createdAt: new Date()
            });

            // Sauvegarder dans le localStorage
            this.saveToLocalStorage();

            this.newTaskTitle = '';
        },
        getTasksByStatus(status) {
            return this.tasks.filter(task => task.status === status);
        },
        moveTask(taskId, newStatus) {
            const taskIndex = this.tasks.findIndex(task => task.id === taskId);
            if (taskIndex !== -1) {
                this.tasks[taskIndex].status = newStatus;
                this.saveToLocalStorage();
            }
        },
        openDeleteModal(taskId) {
            this.taskToDeleteId = taskId;
            this.showDeleteModal = true;
        },
        closeDeleteModal() {
            this.showDeleteModal = false;
            this.taskToDeleteId = null;
        },
        confirmDelete() {
            if (this.taskToDeleteId !== null) {
                this.tasks = this.tasks.filter(task => task.id !== this.taskToDeleteId);
                this.saveToLocalStorage();
                this.closeDeleteModal();
            }
        },
        getNextStatus(currentStatus) {
            const statusOrder = ['todo', 'inProgress', 'done'];
            const currentIndex = statusOrder.indexOf(currentStatus);
            return statusOrder[currentIndex + 1] || currentStatus;
        },
        saveToLocalStorage() {
            // Convertir les dates en chaînes pour le stockage
            const tasksToStore = this.tasks.map(task => ({
                ...task,
                createdAt: task.createdAt.toString()
            }));

            localStorage.setItem('tasks', JSON.stringify(tasksToStore));
        },
        loadFromLocalStorage() {
            const storedTasks = localStorage.getItem('tasks');
            if (storedTasks) {
                // Reconvertir les chaînes en objets Date
                this.tasks = JSON.parse(storedTasks).map(task => ({
                    ...task,
                    createdAt: new Date(task.createdAt)
                }));
            }
        },
        // Méthodes pour le drag and drop
        onDragStart(event, taskId) {
            this.draggedTaskId = taskId;
            event.dataTransfer.effectAllowed = 'move';
            // Ajouter une classe pour le style pendant le drag
            event.target.classList.add('dragging');
        },
        onDragEnd(event) {
            // Retirer la classe de style
            event.target.classList.remove('dragging');
        },
        onDragOver(event, columnId) {
            event.preventDefault(); // Nécessaire pour permettre le drop
            // Ajouter une classe pour indiquer que la colonne peut recevoir le drop
            const column = event.currentTarget;
            column.classList.add('drag-over');
        },
        onDragLeave(event) {
            // Retirer la classe quand on quitte la zone de drop
            event.currentTarget.classList.remove('drag-over');
        },
        onDrop(event, columnId) {
            event.preventDefault();
            // Retirer la classe de style
            event.currentTarget.classList.remove('drag-over');

            // Déplacer la tâche vers la nouvelle colonne
            if (this.draggedTaskId) {
                this.moveTask(this.draggedTaskId, columnId);
                this.draggedTaskId = null;
            }
        }
    }
}
</script>

<template>
    <div class="space-y-6">
        <div class="bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs shadow rounded-lg p-6">
            <h2 class="text-xl font-semibold text-gray-800 mb-6">Planification de Tâches</h2>

            <div class="mb-6">
                <div class="flex space-x-4 mb-4">
                    <input v-model="newTaskTitle" type="text"
                        class="flex-1 pl-2 rounded-md bg-white/40 border-gray-300 shadow-sm focus:outline-primary-400"
                        placeholder="Nouvelle tâche" />
                    <button @click="addTask"
                        class="px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700">
                        Ajouter
                    </button>
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <div v-for="column in taskColumns" :key="column.id" class="bg-white/60 p-4 rounded-lg min-h-[200px]"
                    @dragover="onDragOver($event, column.id)" @dragleave="onDragLeave($event)"
                    @drop="onDrop($event, column.id)">
                    <h3 class="font-medium text-gray-700 mb-3">{{ column.name }}</h3>
                    <div class="space-y-2">
                        <div v-for="task in getTasksByStatus(column.id)" :key="task.id"
                            class="bg-white p-3 rounded shadow-sm cursor-move" draggable="true"
                            @dragstart="onDragStart($event, task.id)" @dragend="onDragEnd($event)">
                            <div class="flex justify-between items-start">
                                <span class="font-medium">{{ task.title }}</span>
                                <div class="flex space-x-1">
                                    <button v-if="column.id !== 'done'"
                                        @click="moveTask(task.id, getNextStatus(column.id))"
                                        class="text-xs bg-gray-200 hover:bg-gray-300 px-2 py-1 rounded">
                                        →
                                    </button>
                                    <button @click="openDeleteModal(task.id)"
                                        class="text-xs bg-red-100 hover:bg-red-200 text-red-600 px-2 py-1 rounded">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none"
                                            viewBox="0 0 24 24" stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                                        </svg>
                                    </button>
                                </div>
                            </div>
                        </div>
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
                    Êtes-vous sûr de vouloir supprimer cette tâche ? Cette action est irréversible.
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
    </div>
</template>

<style scoped>
/* Styles pour le drag and drop */
.dragging {
    opacity: 0.5;
    transform: scale(0.95);
}

.drag-over {
    background-color: rgba(243, 244, 246, 0.8);
    border: 2px dashed #d1d5db;
}

/* Ajouter un curseur de déplacement pour indiquer que l'élément est glissable */
[draggable="true"] {
    cursor: grab;
}

[draggable="true"]:active {
    cursor: grabbing;
}
</style>
