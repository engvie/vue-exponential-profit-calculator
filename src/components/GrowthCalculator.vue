'<template>
  <div>
      <form>
        <div class="form-group">
          <label>Startsumma:</label>
          <input type="number" v-model="start" class="form-control" min="1">
        </div>
        <div class="form-group">
          <label>Procent:</label>
          <input type="number" v-model="percentage" class="form-control" min="1" placeholder="%">
        </div>
        <div class="form-group">
          <label>Steg:</label>
          <input type="number" v-model="steps" class="form-control" min="1">
        </div>
        <div class="form-group">
          <label>Visnings, stegning:</label>
          <input type="number" v-model="displaySteps" class="form-control" min="1">
        </div>
      </form>

      <div class="chart">
        <line-chart
          :chart-data="{
            labels: result.stepNumbers,
            datasets: [
              {
                label: 'Start '+ start +' | '+ percentage + '% | ' + steps + ' steps',
                borderColor: '#007bff',
                fill: false,
                data: result.chart,
              },
            ],
          }"
        />
      </div>

      <div class="result">
        <ul>
          <li
            v-for="(item, i) in result.steps"
            :key="i"
          >
            <span class="step-nr">{{ item.step }}:</span> <span class="step-sum">{{ item.sum }}</span>
          </li>
        </ul>
      </div>

  </div>
</template>

<script>
import LineChart from './LineChart.vue';

export default {
  components: {
    LineChart,
  },
  data: () => {
    return {
      start: 10000,
      percentage: 2,
      steps: 100,
      displaySteps: 1,
      current: 0,
    };
  },
  computed: {
    convertedPercentage: function() {
      return 1 + parseFloat(this.percentage / 100);
    },
    result: function() {
      let result = {
        all: [],
        steps: [],
        stepNumbers: [],
        chart: [],
      };
      let current = this.start;
      let displaySteps = parseInt(this.displaySteps) - 1;
      for (let i = 1; i <= this.steps; i++) {
        current = current * this.convertedPercentage;
        let item = {
          step: i,
          sum: this.formatNumber(
            Math.round(current)
          )
        };
        result.all.push(item);
        result.chart.push(Math.round(current));
        result.stepNumbers.push(i);
        
        if (i == 1 || i > displaySteps) {
          result.steps.push(item);
          displaySteps += this.displaySteps;
        }
      }

      return result;
    },
  },
  methods: {
    formatNumber: function(num) {
      return num.toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1\'')
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
label {
  font-weight: 600;
  margin: 20px 0;
}
input {
  text-align: center;
  font-weight: 600;
  height: 50px;
}
.result {
  ul {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    li {
      list-style: none;
      text-align: center;
      margin: 10px 0;
      width: 25%;
      .step-nr {
        display: inline-block;
        min-width: 100px;
        text-align: center;
      }
      .step-sum {
        display: inline-block;
        font-weight: 600;
      }
    }
  }
}
</style>
