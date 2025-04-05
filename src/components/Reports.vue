<script>
export default {
    name: 'Reports',
    props: {
        timeEntries: {
            type: Array,
            required: true
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
        formatTime(ms) {
            const seconds = Math.floor((ms / 1000) % 60);
            const minutes = Math.floor((ms / (1000 * 60)) % 60);
            const hours = Math.floor(ms / (1000 * 60 * 60));

            return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
        }
    }
}
</script>

<template>
    <div class="space-y-6">
        <div class="bg-white/40 bg-clip-padding backdrop-filter backdrop-blur-xs shadow rounded-lg p-6">
            <h2 class="text-xl font-semibold text-gray-800 mb-6">Rapports d'Activité</h2>

            <div class="mb-6">
                <h3 class="text-lg font-medium text-gray-700 mb-2">Résumé</h3>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div class="bg-white/40 p-4 rounded-lg">
                        <div class="text-sm text-gray-500">Total des heures</div>
                        <div class="text-2xl font-semibold">{{ formatTime(totalTime) }}</div>
                    </div>
                    <div class="bg-white/40 p-4 rounded-lg">
                        <div class="text-sm text-gray-500">Entrées</div>
                        <div class="text-2xl font-semibold">{{ timeEntries.length }}</div>
                    </div>
                    <div class="bg-white/40 p-4 rounded-lg">
                        <div class="text-sm text-gray-500">Moyenne par entrée</div>
                        <div class="text-2xl font-semibold">{{ formatTime(averageTime) }}</div>
                    </div>
                </div>
            </div>

            <div>
                <h3 class="text-lg font-medium text-gray-700 mb-2">Détails par activité</h3>
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-white/40">
                            <tr>
                                <th
                                    class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Activité
                                </th>
                                <th
                                    class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Temps
                                    total</th>
                                <th
                                    class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Entrées
                                </th>
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200">
                            <tr v-for="(report, activity) in activityReports" :key="activity">
                                <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{{ activity }}
                                </td>
                                <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{
                                    formatTime(report.totalTime) }}
                                </td>
                                <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{ report.count }}</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>