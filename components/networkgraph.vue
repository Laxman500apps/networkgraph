<template>
  <div
    @wheel.prevent="handleMouseWheel"
    @mousedown.prevent="startPan"
    @mousemove="pan"
    @mouseup="stopPan">
    <highchart
      :style="{ transform: `scale(${zoom}) translate(${panX}px, ${panY}px)` }"
      :options="chartOptions"
      :modules="[chartData.chart.type]"></highchart>
  </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps(["chartData"]);
const zoom = ref(1);
const isPanning = ref(false);
const panStartX = ref(0);
const panStartY = ref(0);
const panX = ref(0);
const panY = ref(0);

const chartOptions = ref({
  chart: props.chartData.chart,
  title: props.chartData.title,
  subtitle: props.chartData.subtitle,

  plotOptions: {
    networkgraph: {
      keys: props.chartData.keys,
      layoutAlgorithm: {
        enableSimulation: props.chartData.enableSimulation,
        linkLength: props.chartData.linkLength,
        friction: props.chartData.friction,

        initialPositions: function () {
          const chart = this.series[0].chart,
            width = chart.plotWidth,
            height = chart.plotHeight;
          this.nodes.forEach(function (node) {
            // If initial positions were set previously, use that

            // positions. Otherwise use random position:

            node.plotX =
              node.plotX === undefined ? Math.random() * width : node.plotX;
            node.plotY =
              node.plotY === undefined ? Math.random() * height : node.plotY;
          });
        },
      },
    },
  },

  series: props.chartData.series,
});

const handleMouseWheel = (event) => {
  // Adjust the zoom level based on the direction of the mouse wheel
  zoom.value += event.deltaY > 0 ? -0.1 : 0.1;

  // Ensure the zoom level stays within a certain range (optional)
  zoom.value = Math.min(Math.max(this.zoom, 0.5), 3);
};

const startPan = (event) => {
  isPanning.value = true;
  panStartX.value = event.clientX;
  panStartY.value = event.clientY;
};

const pan = (event) => {
  if (isPanning.value) {
    panX.value += event.clientX - panStartX.value;
    panY.value += event.clientY - panStartY.value;
    panStartX.value = event.clientX;
    panStartY.value = event.clientY;
  }
};

const stopPan = () => {
  isPanning.value = false;
};
</script>
