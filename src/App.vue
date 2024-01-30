<template>
  <div class="tanks">
    <Tank
      v-for="(tank, index) in tanks"
      :key="index"
      :level="tank.level"
      :buffer="tank.buffer"
      @empty-tank="emptyTank(index)"
      @start-filling="startFilling(index)"
      @stop-filling="stopFilling(index)"
    />
  </div>
</template>

<script setup>
import { reactive, onUnmounted } from "vue";
import Tank from "./components/Tank.vue";

const tanks = reactive([
  { id: 1, level: 0, buffer: 0, addIntervalId: null },
  { id: 2, level: 0, buffer: 0, addIntervalId: null },
  { id: 3, level: 0, buffer: 0, addIntervalId: null },
  { id: 4, level: 0, buffer: 0, addIntervalId: null },
]);

let addWaterToTanksInterval = setInterval(() => distributeWater(tanks), 50);

const startFilling = (index) => {
  console.log("startFilling", index, tanks);
  if (!addWaterToTanksInterval) {
    addWaterToTanksInterval = setInterval(() => distributeWater(tanks), 50);
  }
  const tank = tanks[index];
  console.log("startFilling tank", tank);
  if (!tank.addIntervalId) {
    tank.addIntervalId = setInterval(() => {
      const intervalSeconds = 20 / 1000;
      const inflowRatePerSecond = 20;
      const addAmount = (inflowRatePerSecond * intervalSeconds) / 100;
      tank.buffer = tank.buffer + addAmount * 100;
    }, 20);
  }
};

const stopFilling = (index) => {
  console.log("stopFilling", index, tanks);
  const tank = tanks[index];
  if (tank.addIntervalId) {
    console.log("stopFilling tank", tank);
    clearInterval(tank.addIntervalId);
    tank.addIntervalId = null;
  }
};

const emptyTank = (index) => {
  tanks[index].level = 0;
  if (!addWaterToTanksInterval) {
    addWaterToTanksInterval = setInterval(() => distributeWater(tanks), 50);
  }
};

// onMounted(() => {
// });

onUnmounted(() => {
  // if (distributionInterval) {
  //   clearInterval(distributionInterval);
  // }
  if (addWaterToTanksInterval) {
    clearInterval(addWaterToTanksInterval);
    addWaterToTanksInterval = null;
  }
});

// const addToTank = (tanks) => {
//   const intervalSeconds = 10 / 1000;
//   const inflowRatePerSecond = 2.5;
//   const addAmount = (100 * inflowRatePerSecond * intervalSeconds) / 100;
//   tanks.forEach((tank) => {
//     if (tank.buffer > 0 && tank.level < 100 - addAmount) {
//       console.log("addToTank", tank);
//       const finalAddAmount = Math.min(tank.buffer, addAmount);
//       tank.level = Math.min(tank.level + finalAddAmount, 100);
//       tank.buffer -= finalAddAmount;
//     }
//   });
//   distributeWater(tanks);
// };

const distributeWater = (tanks) => {
  const intervalSeconds = 50 / 1000;
  const outflowRatePerSecond = 2.5;
  const flowAmount = (100 * (outflowRatePerSecond * intervalSeconds)) / 100;
  console.log("flowAmount", flowAmount);

  tanks.forEach((tank) => {
    if (tank.buffer > 0 && tank.level < 100 - flowAmount) {
      console.log("addToTank", tank);
      const finalAddAmount = Math.min(tank.buffer, flowAmount);
      tank.level = Math.min(tank.level + finalAddAmount, 100);
      tank.buffer -= finalAddAmount;
    }
  });

  let totalWater = tanks.reduce((acc, tank) => acc + tank.level, 0);
  console.log("totalWater", totalWater);

  const averageLevel = totalWater / tanks.length;
  console.log("averageLevel", averageLevel);
  const isEquilibrium = tanks.every(
    (tank) => Math.abs(tank.level - averageLevel) < 0.0001
  );
  if (isEquilibrium) {
    console.log("Equilibrium reached:", tanks);
    clearInterval(addWaterToTanksInterval);
    addWaterToTanksInterval = null;
    return;
  }

  let totalOutflow = 0;
  tanks.forEach((tank) => {
    if (tank.level > averageLevel) {
      totalOutflow += flowAmount;
      tank.level -= flowAmount;
    }
  });

  const totalDeficit = tanks.reduce((acc, tank) => {
    return tank.level < averageLevel ? acc + (averageLevel - tank.level) : acc;
  }, 0);
  tanks.forEach((tank) => {
    if (tank.level < averageLevel) {
      const deficitProportion = (averageLevel - tank.level) / totalDeficit;
      console.log("deficitProportion", tank.id, deficitProportion);
      const inflow = totalOutflow * deficitProportion;
      console.log("inflow", tank.id, inflow);
      tank.level += inflow;
    }
  });
};

// const distributionInterval = setInterval(() => distributeWater(tanks), 200);

</script>

<style>
.tanks {
  display: flex;
  min-width: fit-content;
  justify-content: center;
  margin: 20px auto;
}
</style>
