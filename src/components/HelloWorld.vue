<template>
  <div>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-dialog
          ref="dialog"
          v-model="modal"
          :return-value.sync="date"
          persistent
          width="290px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="date"
              label="Picker in dialog"
              prepend-icon="mdi-calendar"
              readonly
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker v-model="date" scrollable range>
            <v-spacer></v-spacer>
            <v-btn text color="primary" @click="modal = false"> Cancel </v-btn>
            <v-btn text color="primary" @click="onClickdate(date)"> OK </v-btn>
          </v-date-picker>
        </v-dialog>
      </v-col>
    </v-row>
    <v-data-table
      v-if="tabledata"
      dense
      :headers="headers"
      :items="tabledata"
      item-key="name"
      class="elevation-1"
    ></v-data-table>
    <Barchart
      v-if="chartdata.labels"
      :chartdata="chartdata"
      :options="options"
    />
  </div>
</template>

<script>
import moment from "moment";
import Barchart from "./Barchart";
export default {
  components: {
    Barchart,
  },
  data: () => ({
    date: null,
    modal: false,
    reportCovid: null,
    tabledata: null,
    headers: [
      { text: "date", value: "date" },
      { text: "cases", value: "cases" },
      { text: "deaths", value: "deaths" },
      { text: "recovered", value: "recovered" },
    ],
    label: [
      { name: "cases", color: "#ff0000" },
      { name: "deaths", color: "#000000" },
      { name: "recovered", color: "#00ff00" },
    ],
    chartdata: {
      labels: null,
      datasets: null,
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
    },
  }),
  methods: {
    async onClickdate(date) {
      this.modal = false;
      const startDate = moment(date[0], "YYYY-MM-DD").format("DD/MM/YYYY");
      const endDate = moment(date[1], "YYYY-MM-DD").format("DD/MM/YYYY");
      const data = await this.$axios({
        method: "get",
        url: "https://disease.sh/v3/covid-19/historical/all?lastdays=30",
      }).then((res) => res.data);
      const { cases, deaths, recovered } = data;
      const key = Object.keys(cases);
      const casesValue = Object.values(cases);
      const recoveredValue = Object.values(recovered);
      const deathsValue = Object.values(deaths);
      const array = key
        .map((item, index) => {
          return {
            date: moment(item, "MM/DD/YYYY").format("DD/MM/YYYY"),
            cases: casesValue[index],
            deaths: deathsValue[index],
            recovered: recoveredValue[index],
          };
        })
        .filter((x) => {
          if (x.date >= startDate && x.date <= endDate) {
            return {
              date: x.date,
              cases: x.cases,
              deaths: x.deaths,
              recovered: x.deaths,
            };
          }
        });
      this.tabledata = array;
      let labels = [];
      array.map((item) => {
        labels.push(item.date);
        return;
      });
      this.chartdata.labels = labels;
      const setData = this.label.map((item) => {
        const datasetObject = {
          label: item.name,
          backgroundColor: item.color,
          data: [],
        };
        array.map((value) => {
          if (item.name === "cases") {
            datasetObject.data.push(value.cases);
          } else if (item.name === "deaths") {
            datasetObject.data.push(value.deaths);
          } else {
            datasetObject.data.push(value.recovered);
          }
          return;
        });
        return datasetObject;
      });
      this.chartdata.datasets = setData;
    },
  },
};
</script>
