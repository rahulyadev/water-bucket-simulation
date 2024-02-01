<template>
  <div class="container">
    <div class="buttons">
      <button
      class="addButton"
        @mousedown="startFilling"
        @mouseup="stopFilling"
        @mouseleave="stopFilling"
        @touchstart="startFilling"
        @touchend="stopFilling"
      >
        ADD
      </button>
      <button class="emptyButton" @click="emitEmpty">EMPTY</button>
    </div>
    <div class="buffer">Water in buffer: {{ (buffer * 10).toFixed(1) }}L</div>
    <div class="tank">
      <div class="water" :style="{ height: waterHeight + 'px', transition: 'height ' + interval + 'ms linear' }"></div>
    </div>
    Capacity: 1000L
    <br />
    Water filled: {{ (level * 10).toFixed(2) }}L
  </div>
</template>

<script setup>
import { computed, toRefs } from "vue";
const props = defineProps({
  level: Number,
  buffer: Number,
  interval: Number,
});
const emit = defineEmits(['start-filling', 'stop-filling', 'empty-tank']);
const { level, buffer, interval } = toRefs(props);

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
  flex-direction: column;
  justify-content: space-between;
  margin: 10px 0;
}

.addButton {
  width: 90px;
  height: 27px;
  font-size: 12px;
  font-weight: bold;
  color: white;
  margin: 5px 0px;
  background-color: rgb(1, 145, 1);
  border: none;
  border-radius: 5px;
  padding: auto 20px;
}

.emptyButton {
  width: 90px;
  height: 27px;
  font-size: 12px;
  font-weight: bold;
  color: red;
  margin: 5px 0px;
  background-color: #fff;
  border: 2px red solid;
  border-radius: 5px;
  padding: auto 20px;
}

.buffer {
  margin: 0px 0 20px 0;
}

.tank {
  width: 175px;
  height: 200px;
  border: 5px grey solid;
  border-radius: 20px;
  position: relative;
  background-color: #fff;
  overflow: hidden;
}

.water {
  background-color: #0077ff;
  width: 165px;
  position: absolute;
  border: 5px #4f77a5 solid;
  border-radius: 14px;
  /* border-bottom-right-radius: 14px; */
  bottom: 0;
}
</style>
