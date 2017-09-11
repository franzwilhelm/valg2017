<template>
  <div v-if="resultater">
    <div>
      <!-- <div id="left">Left side div</div>
      <div id="right">Right side div</div> -->
      <img class="center" src="http://www.vg.no/spesial/2017/valg/img/dukker_valgomat.gif">
      <chartist v-if="update" class="lines-bars" :type="'Line'" :data="tot" :options="linesOptions"></chartist>
      <div>
        <chartist class="lines-bars center" :type="'Pie'" :data="donutData" :options="donutOptions"></chartist>
        <table class="center">
          <th>Graf</th>
          <th>Parti</th>
          <th>Mandater</th>
          <th>Stemmer</th>
          <th>Prosent</th>
          <!-- <chartist class="lines-bars" :type="'Line'" :data="" :options="linesOptions"></chartist> -->
          <tr align="left" v-for="(parti, index) in resultater.data.partier" v-if="index < 9">
            <td>
              <transition name="fade">
            <chartist v-if="update" class="lines-bars" :type="'Line'" :data="ser[index]" :options="linesOptions"></chartist>
          </transition>
            </td>
            <td><strong>{{parti.kortNavn}}</strong></td>
            <td>{{parti.mandater.prognose.antall}}
              <strong :style="parti.mandater.prognose.endringAntall > 0 ? 'color:red':'color:green'">
                ({{(parti.mandater.prognose.endringAntall > 0 ? '+' : '') + parti.mandater.prognose.endringAntall}})
              </strong>
            </td>
            <td>{{parti.stemmer.antall}}</td>
            <td>{{parti.stemmer.prognose.prosent}}%</td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Chartist from 'vue-bulma-chartist';
import Vue from 'vue';

export default {
  name: 'hello',
  components: {
    Chartist,
  },
  data() {
    return {
      resultater: null,
      donutOptions: {
        donut: true,
      },
      linesOptions: {
        fullWidth: true,
        chartPadding: {
          right: 40,
        },
      },
      update: false,
      series: [],
      total: [],
    };
  },
  mounted() {
    this.oppdaterResultater();
    setInterval(() => {
      this.oppdaterResultater();
    }, 12000);
  },
  methods: {
    async oppdaterResultater() {
      this.update = false;
      const response = await axios('https://valg-api.nrk.no/valg/resultater/2017/st/res/0');
      this.update = true;
      console.log(response);
      this.resultater = response;
    },
  },
  computed: {
    donutData() {
      const partier = this.resultater.data.partier;
      const donut = {
        series: [],
        labels: [],
      };
      for (let i = 0; i < partier.length; i += 1) {
        if (i < 9) {
          const prosent = partier[i].stemmer.antall / this.resultater.data.totaltAntallStemmer;
          console.log(prosent);
          if (!this.series[i]) {
            this.series[i] = { series: [[prosent]] };
          } else {
            console.log(prosent);
            const now = this.series[i].series[0];
            Vue.set(this.series, i, { series: [now.concat(prosent)] });
          }
          donut.labels.push(partier[i].kortNavn);
          donut.series.push(partier[i].stemmer.antall);
        }
      }
      return donut;
    },
    ser() {
      return this.series;
    },
    tot() {
      this.total = [...this.total, this.resultater.data.opptaltProsent];
      return { series: [this.total] };
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#left {
  float: left;
  width: 50%;
  overflow: hidden;
}

#right {
  overflow: hidden;
}
.lines-bars {
  height: 300px;
}
.center {
  margin: auto;
  padding: 10px;
}
table {
    border-collapse: collapse;
}

table, th, td {
  margin: 0px;
    border: 1px solid black;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 2s
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
}
</style>
