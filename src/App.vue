<script setup>
import { ref, shallowRef, computed, watch, nextTick, onMounted } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);
const canvas = ref(null);
const chart = shallowRef(null);
const input = ref(60.0);
const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});
const addWeight = () => {
  weights.value.push({
    weight: input.value,
    date: new Date().getTime(),
  });
};
watch(
  weights,
  (newVal) => {
    const cloneWeights = [...newVal];

    if (chart.value) {
      chart.value.data.labels = cloneWeights
        .sort((a, b) => a.date - b.date)
        .map((w) => new Date(w.date).toLocaleDateString())
        .slice(-7);

      chart.value.data.datasets[0].data = cloneWeights
        .sort((a, b) => a.date - b.date)
        .map((w) => w.weight)
        .slice(-7);

      chart.value.update();
      return;
    }
    nextTick(() => {
      chart.value = new Chart(canvas.value.getContext("2d"), {
        type: "line",
        data: {
          labels: cloneWeights
            .sort((a, b) => a.date - b.date)
            .map((w) => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              label: "Weight",
              data: cloneWeights
                .sort((a, b) => a.date - b.date)
                .map((w) => w.weight),
              backgroundColor: "rgba(47, 117, 209,0.2)",
              borderColor: "rgb(47, 117, 209)",
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
      <span> {{ currentWeight.weight }}</span>
      <small> Current weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="input" />
      <input type="submit" class="button" value="Add weight" />
    </form>

    <div v-if="weights && weights.length > 0">
      <div class="title">Last 7 days</div>
      <div class="canvas-box">
        <canvas ref="canvas"></canvas>
      </div>
      <div class="weight-list">
        <div class="title">Weight History</div>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }} kg </span>
            <small> ({{ new Date(weight.date).toLocaleDateString() }}) </small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style>
* {
  font-family: "montserrat", sans-serif;
  box-sizing: border-box;
}

body {
  text-align: center;
  background-color: #eee;
}

main {
  margin: auto;
  height: 500px;
  width: 500px;
  color: #4b4b4b;
}

h1 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
}

.current {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 150px;
  height: 150px;
  text-align: center;
  background-color: white;
  border-radius: 50%;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border: 4px solid #5c8cca;
  margin: auto;
}

.current span {
  display: block;
  font-size: 2rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.current small {
  color: #888;
  font-style: italic;
}

div.title {
  margin: 1rem 0 0.4rem 0;
  text-align: left;
}

form {
  display: flex;
  justify-content: center;
  margin-top: 2rem;
}

input[type="number"] {
  background-color: #fff;
  outline: none;
  border: none;
  border-radius: 0.25rem;
  padding: 0.5rem 1rem;
  margin-right: 1rem;
  font-size: 14px;
  width: 150px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

input[type="submit"] {
  color: #fff;
  background-color: #2f75d1;
  outline: none;
  border: none;
  cursor: pointer;
  font-weight: bold;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  opacity: 0.7;
  transition: 0.3s;
}

input[type="submit"]:hover {
  opacity: 1;
}

.canvas-box {
  margin-bottom: 2rem;
}

canvas {
  background-color: #fff;
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  padding: 0.5rem;
}

.weight-list ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.weight-list ul li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem;
  cursor: pointer;
}

.weight-list ul li:nth-child(even) {
  background-color: #dfdfdf;
}

.weight-list ul li:hover {
  background-color: #f8f8f8;
}

.weight-list ul li span {
  font-size: 16px;
  font-weight: bold;
}
</style>
