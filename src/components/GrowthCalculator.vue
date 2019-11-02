<template>
  <div>
    <div class="row">
      <div class="col-lg-9">
        <form @submit.prevent="save">
          <div class="row">
            <div class="col-lg-6">
              <div class="form-group">
                <label>Startsumma:</label>
                <input type="number" v-model="form.start" class="form-control" min="1">
              </div>
            </div>
            <div class="col-lg-6">
              <div class="form-group">
                <label>Procent:</label>
                <input type="number" v-model="form.percentage" class="form-control" min="1" placeholder="%">
              </div>
            </div>
            <div class="col-lg-6">
              <div class="form-group">
                <label>Steg:</label>
                <input type="number" v-model="form.steps" class="form-control" min="1">
              </div>
            </div>
            <div class="col-lg-6">
              <div class="form-group">
                <label>Visnings, stegning:</label>
                <input type="number" v-model="form.displaySteps" class="form-control" min="1">
              </div>
            </div>
            <div class="col-lg-12">
              <button type="submit" class="btn btn-primary w-100">Spara <span v-if="currentStorageKey != null">[{{ currentStorageKey + 1 }}]</span></button>
            </div>
          </div>
        </form>
      </div>
      <div class="col-lg-3">
        <ul class="list-group storage-list">
          <li
            v-for="(item, key) in storage"
            :key="key"
            :class="[
              'list-group-item d-flex',
              'justify-content-between',
              {current: key == currentStorageKey}
            ]"
            @click="setCurrent(key)"
          >
            <span><i class="fas fa-dollar-sign"></i> {{ item.start }}</span>
            <span><i class="fas fa-shoe-prints"></i> {{ item.steps }}</span>
            <span><i class="fas fa-percentage"></i> {{ item.percentage }}</span>
            <span class="number">
              {{ key + 1 }}
            </span>
            <span class="total">
              {{ getTotal(item) }}
            </span>
            <span class="remove" @click="removeStored(key)""><i class="fas fa-trash-alt"></i></span>
          </li>
        </ul>
      </div>
    </div>

    <div class="row mt-5">
      <div class="col-lg-9">
        <div class="chart">
          <line-chart
            :chart-data="{
              labels: result.stepNumbers,
              datasets: [
                {
                  label: 'Start '+ form.start +' | '+ form.percentage + '% | ' + form.steps + ' steps | ' + this.total + ' totalt',
                  borderColor: '#007bff',
                  fill: false,
                  data: result.chart,
                },
              ],
            }"
          />
        </div>
      </div>
      <div class="col-lg-3">
        <h3 class="text-center text-primary">{{ total }}</h3>
        <ul class="list-group">
          <li
            v-for="(item, i) in result.steps"
            :key="i"
            class="list-group-item d-flex"
          >
            <span class="step-nr">{{ item.step }}:</span>
            <span class="step-sum ml-auto">{{ item.sum }}</span>
          </li>
        </ul>
      </div>
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
      storage: [],
      currentStorageKey: null,
      form: {
        start: 10000,
        percentage: 2,
        steps: 100,
        displaySteps: 1,
      },
    };
  },
  computed: {
    convertedPercentage: function() {
      return 1 + parseFloat(this.form.percentage / 100);
    },
    result: function() {
      let result = {
        all: [],
        steps: [],
        stepNumbers: [],
        chart: [],
      };
      let current = this.form.start;
      let displaySteps = parseInt(this.form.displaySteps) - 1;
      for (let i = 1; i <= this.form.steps; i++) {
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
          displaySteps += parseInt(this.form.displaySteps);
        }
      }

      return result;
    },
    total () {
      return this.getTotal(this.form);
    },
  },
  methods: {
    formatNumber: function(num) {
      return Math.round(num).toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1\'')
    },
    getStorage () {
      try {
        return JSON.parse(window.localStorage.getItem('trading') || '[]');
      } catch {
        return [];
      }
    },
    saveStorage (storage) {
      window.localStorage.setItem('trading', JSON.stringify(storage));
      this.setStorage();
    },
    setStorage() {
      this.storage = this.getStorage();
    },
    removeStored (key) {
      let storage = this.getStorage();
      storage.splice(key, 1);
      this.currentStorageKey = null;
      this.saveStorage(storage);
    },
    setCurrent(key) {
      let setKey = key;
      if (this.currentStorageKey != key) {
        let storage = this.getStorage();
        if (storage[key]) {
          this.form = storage[key];
        }
      } else {
        setKey = null;
      }

        this.currentStorageKey = setKey;
        window.localStorage.setItem('currentStorageKey', setKey);
    },
    getTotal(data) {
      return this.formatNumber(data.start * ((1 + parseFloat(data.percentage / 100)) ** data.steps));
    },
    save () {
      let storage = this.getStorage();
      if (this.currentStorageKey != null) {
        storage[ this.currentStorageKey ] = this.form;
      } else {
        this.currentStorageKey = storage.push(this.form) - 1;
      }

      this.saveStorage(storage);
    },
  },
  mounted () {
    this.setStorage();

    let currentKey = window.localStorage.getItem('currentStorageKey');
    if (currentKey != null && this.storage[currentKey]) {
      this.currentStorageKey = parseInt(currentKey);
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
label {
  font-weight: 600;
}
input {
  text-align: center;
  font-weight: 600;
  height: 50px;
}
.storage-list {
  i {
    font-size: 0.8em;
  }
  li {
    position: relative;
    .total {
      position: absolute;
      top: 100%;
      left: 50%;
      transform: translate(-50%, -50%);
      z-index: 2;
      background: #fff;
      font-size: 0.7em;
      padding: 0 15px;
      border: 1px solid #ccc;
      border-top: none;
    }
    .number {
      position: absolute;
      top: 2px;
      left: 3px;
      font-size: 0.7em;
      font-weight: 600;
    }
    .remove {
      position: absolute;
      top: 50%;
      left: 100%;
      transform: translateY(-50%);
      z-index: 2;
      background: #fff;
      padding: 5px;
      border: 1px solid #ccc;
      border-left: none;
      color: red;
      cursor: pointer;
    }
    &.current {
      //border-left: 5px solid var(--primary);
      box-shadow: -5px 0px 0px 0px var(--primary);
      color: var(--primary);
    }
  }
}
</style>
