<script setup lang="ts">
import type { ChartData, ChartOptions } from 'chart.js';
import { defineChartComponent } from 'vue-chart-3';

const FunnelChart = defineChartComponent('funnel', 'funnel');

const chartOptions = ref<ChartOptions<'funnel'>>({
    responsive: true,
    maintainAspectRatio: false,
    interaction: {
        intersect: false,
        mode: 'nearest',
        axis: 'x',
        includeInvisible: true
    },
    scales: {
        y: {
            ticks: { display: true },
            grid: {
                display: true,
                drawBorder: false,
                color: '#CCCCCC22',
                // borderDash: [5, 10]
            },
        },
        x: {
            ticks: { display: true },
            grid: {
                display: true,
                drawBorder: false,
                color: '#CCCCCC22',
            }
        }
    },
    plugins: {
        legend: { display: false },
        title: { display: false },
        tooltip: {
            enabled: true,
            backgroundColor: 'rgba(0, 0, 0, 0.8)',
            titleFont: { size: 16, weight: 'bold' },
            bodyFont: { size: 14 },
            padding: 10,
            cornerRadius: 4,
            boxPadding: 10,
            caretPadding: 20,
            yAlign: 'bottom',
            xAlign: 'center',
        }
    },
});


const chartData = ref<ChartData<'funnel'>>({
    labels: [],
    datasets: [
        {
            data: [],
            backgroundColor: [
                '#5680F877',
                "#6bbbe377",
                "#a6d5cb77",
                "#fae0b977",
                "#f28e8e77",
                "#e3a7e477",
                "#c4a8e177",
                "#8cc1d877",
                "#f9c2cd77",
                "#b4e3b277",
                "#ffdfba77",
                "#e9c3b577",
                "#d5b8d677",
                "#add7f677",
                "#ffd1dc77",
                "#ffe7a177",
                "#a8e6cf77",
                "#d4a5a577",
                "#f3d6e477",
                "#c3aed677"
            ],
            // borderColor: '#0000CC',
            // borderWidth: 4,
            fill: true,
            tension: 0.45,
            pointRadius: 0,
            pointHoverRadius: 10,
            hoverBackgroundColor: '#26262677',
            // hoverBorderColor: 'white',
            // hoverBorderWidth: 2,
        },
    ],
});


onMounted(async () => {

    // const c = document.createElement('canvas');
    // const ctx = c.getContext("2d");
    // let gradient: any = `${'#0000CC'}22`;
    // if (ctx) {
    //     gradient = ctx.createLinearGradient(0, 25, 0, 300);
    //     gradient.addColorStop(0, `${'#0000CC'}99`);
    //     gradient.addColorStop(0.35, `${'#0000CC'}66`);
    //     gradient.addColorStop(1, `${'#0000CC'}22`);
    // } else {
    //     console.warn('Cannot get context for gradient');
    // }

    // chartData.value.datasets[0].backgroundColor = [gradient];

});

const eventsData = useFetch(`/api/data/events`, {
    headers: useComputedHeaders(), lazy: true
});

const enabledEvents = ref<string[]>([]);

async function onEventCheck(eventName: string) {
    const index = enabledEvents.value.indexOf(eventName);
    if (index == -1) {
        enabledEvents.value.push(eventName);
    } else {
        enabledEvents.value.splice(index, 1);
    }


    chartData.value.labels = enabledEvents.value;
    chartData.value.datasets[0].data = [];

    for (const enabledEvent of enabledEvents.value) {
        const target = (eventsData.data.value ?? []).find(e => e._id == enabledEvent);
        chartData.value.datasets[0].data.push(target?.count || 0);
    }
}

</script>


<template>
    <CardTitled title="Funnel"
        sub="Monitor and analyze the actions your users are performing on your platform to gain insights into their behavior and optimize the user experience">
        <div class="flex gap-2 justify-between lg:flex-row flex-col">
            <div class="flex flex-col gap-1">
                <div class="min-w-[20rem] text-lyx-text-darker">
                    Select two or more events
                </div>
                <div class="flex flex-col gap-1">
                    <div v-for="event of eventsData.data.value">
                        <UCheckbox color="secondary" @change="onEventCheck(event._id)"
                            :value="enabledEvents.includes(event._id)" :label="event._id">
                        </UCheckbox>
                    </div>
                </div>
            </div>
            <div class="grow">
                <FunnelChart :chart-data="chartData" :options="chartOptions"> </FunnelChart>
            </div>
        </div>
    </CardTitled>
</template>
