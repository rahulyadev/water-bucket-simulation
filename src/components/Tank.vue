<template>
  <div class="container">
    <div class="buttons">
      <button
        @mousedown="startFilling"
        @mouseup="stopFilling"
        @mouseleave="stopFilling"
        @touchstart="startFilling"
        @touchend="stopFilling"
      >
        Add water
      </button>
      <button @click="emitEmpty">Empty tank</button>
    </div>
    <div class="buffer">Water in buffer: {{ (buffer * 10).toFixed(1) }}L</div>
    <div class="tank">
      <div class="water" :style="{ height: waterHeight + 'px' }"></div>
    </div>
    Capacity: 1000L
    <br />
    Water filled: {{ (level * 10).toFixed(3) }}L
  </div>
</template>

<script setup>
import { computed, toRefs } from "vue";
const props = defineProps({
  level: Number,
  buffer: Number,
});
const emit = defineEmits(['start-filling', 'stop-filling', 'empty-tank']);
const { level, buffer } = toRefs(props);

const waterHeight = computed(() => {
  const tankHeight = 200;
  return (level.value / 100) * tankHeight;
});

const startFilling = () => {
  emit("start-filling");
};

const stopFilling = () => {
  emit("stop-filling");
};

const emitEmpty = () => {
  emit("empty-tank");
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  margin: 5px 5px;
}
.buttons {
  display: flex;
  justify-content: space-between;
  margin: 10px 0;
}

button {
  width: 50px;
  margin: 0 5px;
}

.tank {
  width: 175px;
  height: 200px;
  border: 2px solid #000;
  border-top: none;
  border-radius: 5px;
  position: relative;
  background-color: #fff;
  overflow: hidden;
}

.water {
  background-color: #0077ff;
  width: 100%;
  position: absolute;
  bottom: 0;
  height: 0;
  transition: max-height 50ms linear;
}
</style>
