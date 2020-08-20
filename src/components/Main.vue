<template>
  <div>
    <div class="row" id="header">
      <div class="col">
        <div class="d-flex justify-content-between">
          <div>
            <b-form-select
              v-model="selected"
              :options="selectOptions"
              @change="changeSelectedState"
              size="sm"
              id="selectState"
            ></b-form-select>
          </div>
          <div>{{ latestStatsDate }}</div>
        </div>
      </div>
    </div>
    <div class="row d-flex justify-content-center pt-4" id="all">
      <div class="col-md-6 p-4" id="mainWin">
        <div class="row">
          <div class="col-5">
            <p>Positive</p>
            <p>Hospitalized</p>
            <p>Deaths(total)</p>
            <p>Total Tests</p>
          </div>
          <div class="col">
            <p>
              <b>{{latestData}}</b>
            </p>
            <p>
              <b>{{hospitalizedCurrently}}</b>
            </p>
            <p>
              <b>{{death}}</b>
              <span class="font-weight-lighter">(+{{ deathIncrease }})</span>
            </p>
            <p>
              <b>{{totalTestsViral}}</b>
            </p>
          </div>
        </div>
        <div class="row pt-3">
          <div class="col">
            <div class="form-check form-check-inline">
              <input
                class="form-check-input"
                type="radio"
                name="graphDataOption"
                id="inlineRadio1"
                value="positiveCases"
                v-model="graphOption"
              />
              <label class="form-check-label font-weight-lighter" for="inlineRadio1">Positive cases</label>
            </div>
            <div class="form-check form-check-inline">
              <input
                class="form-check-input"
                type="radio"
                name="graphDataOption"
                id="inlineRadio2"
                value="positiveTested"
                v-model="graphOption"
              />
              <label
                class="form-check-label font-weight-lighter"
                for="inlineRadio2"
              >% positive/tested</label>
            </div>
            <div class="form-check form-check-inline">
              <input
                class="form-check-input"
                type="radio"
                name="graphDataOption"
                id="inlineRadio3"
                value="deathPositive"
                v-model="graphOption"
              />
              <label
                class="form-check-label font-weight-lighter"
                for="inlineRadio3"
              >% death/positive</label>
            </div>
          </div>
        </div>
        <div class="row pt-2">
          <div class="col">
            <bar-chart v-if="dataLoaded" :chartData="chartData" :options="options" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const axios = require("axios");

import BarChart from "../views/BarChart.vue";
import statesIdentifiers from "../js/states.js";

export default {
  name: "Main",
  components: {
    BarChart
  },

  data: function() {
    return {
      dataLoaded: false,

      selected: "ca",  // default..because I live here.
      selectOptions: statesIdentifiers,

      graphOption: "positiveCases",

      latestStatsDate: "",
      latestData: 0,
      hospitalizedCurrently: 0,
      death: 0,
      totalTestsViral: 0,
      deathIncrease: 0,

      chartData: {
        labels: [],
        datasets: [
          {
            label: "Positive cases",
            backgroundColor: "rgb(74, 92, 78)",
            data: []
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: true,
        aspectRatio: 2, // Not working.
        legend: {
            display: false,
        }
      }
    };
  },

  watch: {
    graphOption: function() {
      this.loadStateData();
    }
  },

  methods: {
    /*
     *  Handle change event for (state) selector
     */
    changeSelectedState: function() {
      this.loadStateData();
    },

    /*
     *  Access the api and load result into the appropriate array.
     */
    loadStateData: async function() {
      // clear existing arrays
      this.dataLoaded = false;
      this.chartData.datasets[0].data.length = 0;
      this.chartData.labels.length = 0;
      try {
        let _result = await axios.get(
          "https://api.covidtracking.com/api/v1/states/" +
            this.selected +
            "/daily.json"
        );

        // load the relavent array.
        if (this.graphOption === "positiveCases") {
          this.chartData.datasets[0].label = "Positive Cases"

          _result.data.forEach(element => {
            // skip objects with incomplete data
            if (element.positive && element.lastUpdateEt) {
              this.chartData.datasets[0].data.unshift(element.positive);
              this.chartData.labels.unshift(element.lastUpdateEt.split(" ")[0]);
            }
          });
        } else if (this.graphOption === "positiveTested") {
          this.chartData.datasets[0].label = "% Positive/tested"

          _result.data.forEach( element => {
            // skip objects with incomplete data
            if (element.positive && element.lastUpdateEt) {
              let posVsTested = (element.positive / element.totalTestResults * 100).toFixed(2) ;

              this.chartData.datasets[0].data.unshift(posVsTested);
              this.chartData.labels.unshift(element.lastUpdateEt.split(" ")[0]);
            }
          })
        } else {
          this.chartData.datasets[0].label = "% death/tested"

          _result.data.forEach( element => {
            // skip objects with incomplete data
            if (element.positive && element.lastUpdateEt) {
              let deathVsPositive = (element.death / element.positive * 100).toFixed(2);

              this.chartData.datasets[0].data.unshift(deathVsPositive);
              this.chartData.labels.unshift(element.lastUpdateEt.split(" ")[0]);
            }
          })
        }
        

        // latest stats
        let latestData = _result.data[0];

        this.latestStatsDate = latestData.lastUpdateEt.split(" ")[0];
        this.latestData = latestData.positive || "-";
        this.hospitalizedCurrently = latestData.hospitalizedCurrently || "-";
        this.death = latestData.death || "-";
        this.deathIncrease = latestData.deathIncrease;
        this.totalTestsViral = latestData.totalTestResults || "-";

        // ok to draw chart
        this.dataLoaded = true;
      } catch (err) {
        console.error(err);
      }
    }
  },

  mounted: function() {
    this.loadStateData();
  }
};
</script>

<style scoped>
#all {
  background-color: rgb(33, 87, 45);
}

#mainWin {
  background-color: rgb(240, 240, 240);
}

#header {
  background-color: rgb(22, 22, 22);
  color: rgb(238, 238, 238);
}

#selectState {
  background-color: rgb(22, 22, 22);
  color: rgb(238, 238, 238);
  border: black;
}
</style>