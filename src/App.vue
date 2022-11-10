<script setup lang="ts">
import {
  ref,
  shallowRef,
  computed,
  watch,
  nextTick,
  ComputedRef,
  Ref,
} from "vue";
import Chart from "chart.js/auto";
import { IWeight, IWeightDate } from "./types";

const weights: Ref<IWeightDate[]> = ref([]);

const weightChartElement: Ref<HTMLCanvasElement | null> = ref(null);

const weightChart: Ref<Chart | null> = shallowRef(null);

const weightInput = ref(60.0);

const dateInput = ref("");

const currentWeight: ComputedRef<IWeightDate | IWeight> = computed(() => {
  return (
    weights.value.sort(
      (weightA, weightB) => weightB.date - weightA.date
    )[0] || { weight: 0 }
  );
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: dateInput.value
      ? new Date(dateInput.value).getTime()
      : new Date().getTime(),
  });
};

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights];
    const sortedNewWeights = ws.sort(
      (weightA, weightB) => weightA.date - weightB.date
    );

    if (weightChart?.value?.data) {
      weightChart.value.data.labels = sortedNewWeights
        .map((weight) => new Date(weight.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = sortedNewWeights
        .map((weight) => weight.weight)
        .slice(-7);

      weightChart.value.update();

      return;
    }

    nextTick(() => {
      const canvas = weightChartElement.value;
      const context = canvas?.getContext("2d");

      if (!context) {
        return;
      }

      weightChart.value = new Chart(context, {
        type: "line",
        data: {
          labels: ws
            .sort((weightA, weightB) => weightA.date - weightB.date)
            .map((weight) => new Date(weight.date).toLocaleDateString()),
          datasets: [
            {
              label: "Weight",
              data: ws
                .sort((weightA, weightB) => weightA.date - weightB.date)
                .map((weight) => weight.weight),
              backgroundColor: "rgba(255, 105, 180, 0.2)",
              borderColor: "rgba(255,105,180)",
              borderWidth: 1,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        },
      });
    });
  },
  { deep: true }
);
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>
    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Current weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" v-model="weightInput" />
      <div class="divider"></div>
      <input type="date" v-model="dateInput" />
      <input type="submit" value="+" />
    </form>

    <div v-if="weights?.length">
      <h2>Last 7 days</h2>

      <div class="canvas-box">
        <canvas ref="weightChartElement"></canvas>
      </div>

      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }}kg</span>
            <small>
              {{ new Date(weight.date).toLocaleDateString() }}
            </small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "montserrat", sans-serif;
}
body {
  background-color: #eee;
}
main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 1.5rem;
}
h1 {
  font-size: 2em;
  text-align: center;
  margin-bottom: 2rem;
}
h2 {
  margin-bottom: 1rem;
  color: #888;
  font-weight: 400;
}

.current {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 200px;
  height: 200px;

  text-align: center;
  background-color: white;
  border-radius: 999px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border: 5px solid hwb(330 41% 0%);

  margin: 0 auto 2rem;
}
.current span {
  display: block;
  font-size: 2em;
  font-weight: bold;
  margin-bottom: 0.5rem;
}
.current small {
  color: #888;
  font-style: italic;
}
form {
  display: flex;
  margin-bottom: 2rem;
  border: 1px solid #aaa;
  border-radius: 0.5rem;
  overflow: hidden;
  transition: 200ms linear;
}
form:focus-within,
form:hover {
  border-color: hotpink;
  border-width: 2px;
}
form input[type="number"] {
  appearance: none;
  outline: none;
  border: none;
  background-color: white;
  padding: 0.75rem;
  font-size: 1.25rem;
  flex-shrink: 3;
  text-align: center;
}
form .divider {
  border-left: 1px solid #aaa;
  height: auto;
  width: 0rem;
}

form input[type="date"] {
  appearance: none;
  outline: none;
  border: none;
  background-color: white;
  padding: 0.75rem;
  font-size: 1rem;
  flex-grow: 2;
  flex-shrink: 0;
}
form input[type="submit"] {
  flex-grow: 2;
  appearance: none;
  outline: none;
  border: none;
  cursor: pointer;
  background-color: hotpink;
  padding: 0.75rem;
  color: white;
  font-size: 1.25rem;
  font-weight: 700;
  transition: 200ms linear;
  border-left: 3px solid transparent;
  flex-shrink: 0;
}
form input[type="submit"]:hover {
  background-color: white;
  color: hotpink;
  border-left-color: hotpink;
}

.canvas-box {
  width: 100vw;
  max-width: 720px;
  background-color: white;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}
.weight-history ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
.weight-history ul li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem;
  cursor: pointer;
}
.weight-history ul li:nth-child(even) {
  background-color: #dfdfdf;
}
.weight-history ul li:hover {
  background-color: #f8f8f8;
}
.weight-history ul li:last-of-type {
  border-bottom: none;
}
.weight-history ul li span {
  display: block;
  font-size: 1.25rem;
  font-weight: 700;
  margin-right: 1rem;
}
.weight-history ul li small {
  color: #888;
  font-style: italic;
}
</style>
