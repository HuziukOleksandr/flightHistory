<template>
  <div class="map">
    <p class="time">Time: {{ time }}</p>
    <p class="scale">Scale: 1000m - {{ scaleInMeters }}m</p>
    <p class="north">N 360&deg</p>
    <p class="south">S 180&deg</p>
    <p class="west">W 270&deg</p>
    <p class="east">E 90&deg</p>
    <img src="../assets/plane.svg" alt="plane" class="plane" :style="planeStyle" />
    <button class="button" @click="Start" v-if="buttonVisible">Start</button>
    <button class="button" style="background-color: gray" @click="Reset" v-else>Reset</button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import type { HistoryType } from "@/types/history-type";

const props = withDefaults(
  defineProps<{
    history: HistoryType[];
  }>(),
  {}
);

const mapScale = 500;
const TIME = 20;

const planePosition = ref({ x: 0, y: 0 });
const time = ref<string>("");
const planeAngle = ref<number>(0);
const timeInterval = ref<number>(0);
const buttonVisible = ref(true);
const interval = ref<number | null>(null);
const scaleInMeters = computed(() => 1000 / mapScale);

const planeStyle = computed(() => {
  return {
    transform: `translate(${planePosition.value.x}px, ${planePosition.value.y}px) rotate(${planeAngle.value}deg)`,
  };
});

const movePlane = (direction: number, distance: number) => {
  const radian = ((direction - 90) * Math.PI) / 180;
  planePosition.value.x += Math.cos(radian) * distance;
  planePosition.value.y += Math.sin(radian) * distance;
  planeAngle.value = direction;

  buttonVisible.value = false;
};

const Start = () => {
  let index = 0;
  interval.value = setInterval(() => {
    if (index < props.history.length) {
      const elem = props.history[index];
      time.value = formatDate(elem.timestamp).toISOString().replace("T", " ").split(".")[0];
      if (index + 1 < props.history.length) {
        timeInterval.value =
          (formatDate(props.history[index + 1].timestamp).getTime() -
            formatDate(elem.timestamp).getTime()) /
          1000;
      }
      const distance = ((+elem.speed / 3.6) * timeInterval.value) / mapScale;
      movePlane(+elem.direction, distance);
      index++;
    } else {
      clearInterval(interval.value!);
    }
  }, (TIME / props.history.length) * 1000);
};

const Reset = () => {
  if (interval.value) {
    clearInterval(interval.value);
  }
  planePosition.value.x = 0;
  planePosition.value.y = 0;
  planeAngle.value = 0;
  time.value = "";
  buttonVisible.value = true;
};

function formatDate(timestamp: string) {
  const date = new Date(+timestamp * 1000);
  return date;
}
</script>

<style scoped>
.map {
  width: 1600px;
  height: 800px;
  position: relative;
  background-image: url("../assets/map.png");
  background-repeat: no-repeat;
  background-size: cover;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 2px solid black;
}

.plane {
  width: 24px;
  transition: transform 0.2s ease-in-out;
}

.button {
  padding: 10px 20px;
  background-color: rgb(6, 158, 6);
  border: none;
  border-radius: 12px;
  bottom: 60px;
  cursor: pointer;
}

.button:hover {
  background-color: #4caf50;
}

.north,
.south,
.west,
.east,
.time,
.button,
.scale {
  position: absolute;
  color: white;
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
  font-weight: 400;
}

.time {
  top: 10px;
  left: 10px;
}

.north {
  top: 10px;
}

.south {
  bottom: 10px;
}

.west {
  left: 10px;
  transform: rotate(-90deg);
}

.east {
  right: 10px;
  transform: rotate(-90deg);
}

.scale {
  right: 10px;
  top: 10px;
}
</style>
