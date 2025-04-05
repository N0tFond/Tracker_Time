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
            ]
        }
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
                        class="flex-1 pl-2 rounded-md border-gray-300 shadow-sm focus:outline-primary-400"
                        placeholder="Nouvelle tâche" />
                    <button @click="addTask"
                        class="px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700">
                        Ajouter
                    </button>
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <div v-for="column in taskColumns" :key="column.id" class="bg-white/60 p-4 rounded-lg">
                    <h3 class="font-medium text-gray-700 mb-3">{{ column.name }}</h3>
                    <div class="space-y-2">
                        <div v-for="task in getTasksByStatus(column.id)" :key="task.id"
                            class="bg-white p-3 rounded shadow-sm">
                            <div class="flex justify-between items-start">
                                <span class="font-medium">{{ task.title }}</span>
                                <div class="flex space-x-1">
                                    <button v-if="column.id !== 'done'"
                                        @click="moveTask(task.id, getNextStatus(column.id))"
                                        class="text-xs bg-gray-200 hover:bg-gray-300 px-2 py-1 rounded">
                                        →
                                    </button>
                                    <button @click="deleteTask(task.id)"
                                        class="text-xs bg-red-100 hover:bg-red-200 text-red-600 px-2 py-1 rounded">
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
</template>