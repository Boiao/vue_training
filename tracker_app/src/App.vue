<script>
import Chart from "chart.js/auto";
import { computed, ref, shallowRef, nextTick } from "vue";
export default {
  data() {
    return {
      weights: [],
      weight_input: 70.0,
      chart: shallowRef(null),
    };
  },
  computed: {
    currentWeight() {
      return (
        this.weights.sort((a, b) => {
          return b.timestamp - a.timestamp;
        })[0] || { value: 0 }
      );
    },
  },
  methods: {
    addWeight() {
      this.weights.push({
        value: this.weight_input,
        timestamp: Date.now(),
      });
    },
  },
  watch: {
    weights: {
      handler(newVal) {
        const ws = [...newVal];

        if (this.chart !== null) {
          this.chart.data.labels = ws
            .sort((a, b) => a.timestamp - b.timestamp)
            .map((w) => new Date(w.timestamp).toLocaleDateString())
            .slice(-7);

          this.chart.data.datasets[0].data = ws
            .sort((a, b) => a.timestamp - b.timestamp)
            .map((w) => w.value)
            .slice(-7);

          this.chart.update();

          return;
        }

        nextTick(() => {
          this.chart = new Chart(this.$refs.weightChartEl.getContext("2d"), {
            type: "line",
            data: {
              labels: ws
                .sort((a, b) => a.timestamp - b.timestamp)
                .map((w) => new Date(w.timestamp).toLocaleDateString()),
              datasets: [
                {
                  label: "Weight",
                  data: ws
                    .sort((a, b) => a.timestamp - b.timestamp)
                    .map((w) => w.value),
                  backgroundColor: "rgba(255, 105, 180, 0.2)",
                  borderColor: "rgba(255, 105, 180, 1)", // 'rgba(255, 99, 132, 1)
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
      deep: true,
    },
  },
};
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>
    <div class="current">
      <span>{{ currentWeight.value }}</span>
      <small>Current weight in Kg</small>
    </div>
    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weight_input" />
      <input type="submit" value="Add weight" />
    </form>
    <div v-if="weights && weights.length > 0">
      <h2>Last 7 days</h2>
      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>
      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.value }} <small>Kg</small> </span>
            <small>{{ new Date(weight.timestamp).toLocaleDateString() }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style>
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

  flex: 1 1 0%;
  padding: 1rem 1.5rem;
  font-size: 1.25rem;
}

form input[type="submit"] {
  appearance: none;
  outline: none;
  border: none;
  cursor: pointer;
  background-color: hotpink;

  padding: 0.5rem 1rem;

  color: white;
  font-size: 1.25rem;
  font-weight: 700;
  transition: 200ms linear;
  border-left: 3px solid transparent;
}

form input[type="submit"]:hover {
  background-color: white;
  color: hotpink;
  border-left-color: hotpink;
}

.canvas-box {
  width: 100%;
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
