<template>
  <div class="tanks">
    <Tank
      v-for="(tank, index) in tanks"
      :key="index"
      :level="tank.level"
      :buffer="tank.buffer"
      :interval="interval"
      @empty-tank="emptyTank(index)"
      @start-filling="startFilling(index)"
      @stop-filling="stopFilling(index)"
    />
  </div>
  <div class="interval-control">
    <label for="intervalInput">Interval (ms):</label>
    <input id="intervalInput" v-model="intervalInput" type="number" />
    <button @click="updateIntervals">Update</button>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted, onUnmounted, computed, watch } from "vue";
import Tank from "./components/Tank.vue";

const tanks = reactive([
  { id: 1, level: 0, buffer: 0, addIntervalId: null },
  { id: 2, level: 0, buffer: 0, addIntervalId: null },
  { id: 4, level: 0, buffer: 0, addIntervalId: null },
  { id: 3, level: 0, buffer: 0, addIntervalId: null },
]);

const distribute = ref(false);
const interval = ref(1000);
const intervalInput = ref(1000);
let addWaterToTanksInterval = setInterval(
  () => distributeWater(tanks),
  interval.value
);

const startFilling = (index) => {
  // console.log("startFilling", index, tanks);
  if (!addWaterToTanksInterval) {
    addWaterToTanksInterval = setInterval(
      () => distributeWater(tanks),
      interval.value
    );
  }
  const tank = tanks[index];
  // console.log("startFilling tank", tank);
  if (!tank.addIntervalId) {
    tank.addIntervalId = setInterval(() => {
      const intervalSeconds = interval.value / 1000;
      const inflowRatePerSecond = 20;
      const addAmount = (inflowRatePerSecond * intervalSeconds) / 100;
      tank.buffer = tank.buffer + addAmount * 100;
    }, interval.value);
  }
};

const stopFilling = (index) => {
  // console.log("stopFilling", index, tanks);
  const tank = tanks[index];
  if (tank.addIntervalId) {
    // console.log("stopFilling tank", tank);
    clearInterval(tank.addIntervalId);
    tank.addIntervalId = null;
  }
};

const emptyTank = (index) => {
  tanks[index].level = 0;
  if (!addWaterToTanksInterval) {
    addWaterToTanksInterval = setInterval(
      () => distributeWater(tanks),
      interval.value
    );
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
    distribute.value = false;
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

const hasBufferWater = computed(() =>
  tanks.some((tank) => tank.buffer > 0.0001 && tank.level < 99.9999)
);

const distributeWater = (tanks) => {
  const intervalSeconds = interval.value / 1000;
  const outflowRatePerSecond = 2.5;
  const flowAmount = (100 * (outflowRatePerSecond * intervalSeconds)) / 100;
  console.log("flowAmount", flowAmount);

  tanks.forEach((tank) => {
    if (tank.buffer > 0 && tank.level <= 100) {
      // console.log("addToTank", tank);
      const finalAddAmount = Math.min(
        tank.buffer,
        flowAmount,
        100 - tank.level
      );
      tank.level = Math.min(tank.level + finalAddAmount, 100);
      tank.buffer -= finalAddAmount;
    }
  });

  const totalWater = tanks.reduce((acc, tank) => acc + tank.level, 0);
  const averageLevel = totalWater / tanks.length;

  // Identify tanks above and below the average level
  let excessTanks = tanks.filter((tank) => tank.level > averageLevel);
  let deficitTanks = tanks.filter((tank) => tank.level < averageLevel);

  // Calculate the total excess and total deficit
  const totalExcess = excessTanks.reduce(
    (acc, tank) => acc + Math.min(flowAmount, tank.level - averageLevel),
    0
  );
  const totalDeficit = deficitTanks.reduce(
    (acc, tank) => acc + Math.min(flowAmount, averageLevel - tank.level),
    0
  );

  // The actual amount to distribute is the minimum of total excess and total deficit
  const amountToDistribute = Math.min(totalExcess, totalDeficit);

  // Distribute the water
  excessTanks.forEach((tank) => {
    const tankExcess = Math.min(flowAmount, tank.level - averageLevel);
    const distributionRatio = tankExcess / totalExcess;
    const amountToRemove = amountToDistribute * distributionRatio;
    tank.level -= amountToRemove; // Remove water from excess tanks
  });

  deficitTanks.forEach((tank) => {
    const tankDeficit = Math.min(flowAmount, averageLevel - tank.level);
    const distributionRatio = tankDeficit / totalDeficit;
    const amountToAdd = amountToDistribute * distributionRatio;
    tank.level += amountToAdd; // Add water to deficit tanks
  });

  // const totalDifference = tanks.reduce((acc, tank) => acc + Math.abs(tank.level - averageLevel), 0);

  // tanks.forEach(tank => {
  //   if (tank.level > averageLevel) {
  //     const difference = tank.level - averageLevel;
  //     const proportion = difference / totalDifference;
  //     const adjustment = proportion * flowAmount;
  //     tank.level = Math.max(averageLevel, tank.level - adjustment);
  //   } else if (tank.level < averageLevel) {
  //     const difference = averageLevel - tank.level;
  //     const proportion = difference / totalDifference;
  //     const adjustment = proportion * flowAmount;
  //     tank.level = Math.min(averageLevel, tank.level + adjustment);
  //   }
  // });

  const isEquilibrium = tanks.every(
    (tank) => Math.abs(tank.level - averageLevel) < 0.0001
  );

  console.log(tanks, hasBufferWater.value, isEquilibrium);

  if (distribute.value) {
    if (isEquilibrium && !hasBufferWater.value) {
      clearInterval(addWaterToTanksInterval);
      addWaterToTanksInterval = null;
      // console.log("Equilibrium reached:", tanks);
      distribute.value = false;
    }
  } else {
    distribute.value = !distribute.value;
  }
};

// watch(hasBufferWater, async (newValue: QuasarChoice, oldValue: QuasarChoice) => {
//   console.log('Watch hasBufferWater ==>>  ', newValue, oldValue);
//   if (hasBufferWater) {
//     loadingAgency.value = true;
//     certificateAgencyChoices.value = await supplierStore.getCertificateAgencies(
//       newValue.value,
//       props.country,
//     );
//     loadingAgency.value = false;
//   }
// });

const updateIntervals = () => {
  interval.value = intervalInput.value;
  clearInterval(addWaterToTanksInterval);
  addWaterToTanksInterval = setInterval(
    () => distributeWater(tanks),
    interval.value
  );
  tanks.forEach((tank) => {
    if (tank.addIntervalId) {
      tank.addIntervalId = setInterval(() => {
        clearInterval(tank.addIntervalId);
        const intervalSeconds = interval.value / 1000;
        const inflowRatePerSecond = 20;
        const addAmount = (inflowRatePerSecond * intervalSeconds) / 100;
        tank.buffer = tank.buffer + addAmount * 100;
      }, interval.value);
    }
  });
};
</script>

<style>
.tanks {
  display: flex;
  min-width: fit-content;
  justify-content: center;
  margin: 20px auto;
}

.interval-control {
  text-align: center;
  margin-top: 20px;
}

.interval-control label {
  margin-right: 10px;
}

.interval-control input {
  width: 100px;
  padding: 5px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

.interval-control button {
  padding: 5px 10px;
  margin-left: 10px;
  margin-top: 200px;
  border: none;
  border-radius: 5px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

.interval-control button:hover {
  background-color: #0056b3;
}
</style>
