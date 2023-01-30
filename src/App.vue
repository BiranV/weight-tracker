<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
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
      <h2>Last 7 days</h2>
      <div class="canvas-box">
        <canvas ref="canvas"></canvas>
      </div>
      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span >{{ weight.weight }} kg </span>
            <small>
               ({{ new Date(weight.date).toLocaleDateString() }})
            </small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style>
* {
  font-family: "montserrat", sans-serif;
  background-color: #eee;
}

body {
  text-align: center;
}

main {
  margin: auto;
  height: 500px;
  width: 500px;
}
.App {
  color: #676767;
  text-align: center;
  padding-top: 4rem;
}

h1 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  color: #4b4b4b;
}
h2 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: #4b4b4b;
}
form {
  display: flex;
  justify-content: center;
  margin-top: 2rem;
}
input[type="number"] {
  outline: none;
  border: none;
  background-color: #fff;
  border-radius: 0.25rem;
  padding: 0.5rem 1rem;
  margin-right: 1rem;
  font-size: 14px;
  width: 150px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}
input[type="submit"] {
  outline: none;
  border: none;
  cursor: pointer;
  color: #fff;
  font-weight: bold;
  text-transform: uppercase;

  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  background-color: #2f75d1;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);

  opacity: 0.7;
  transition: 0.3s;
}
input[type="submit"]:hover {
  opacity: 1;
}

.canvas-box{
  margin-bottom: 2rem;
}

ul {
  margin:auto;
  list-style: none;
    overflow: auto;
    height: 200px;
    width: 250px;
    padding: 0;
   
}

li {
  line-height: 1.5;
  display: flex;
  justify-content: space-between;
}


</style>
