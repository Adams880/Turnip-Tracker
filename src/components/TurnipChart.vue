<template>
  <div class="chart">
    <apexchart
      width = "1900"
      height = "800"
      type = "line"
      :options = "options"
      :series = "series"
    ></apexchart>
  </div>
</template>

<script>
import VueApexCharts from 'vue-apexcharts';
// import VueAxios from 'vue-axios';
export default {
  name: 'TurnipChart',
  components: {
    apexchart: VueApexCharts,
    // axios: VueAxios
  },
  data: function() {
    return {
      idToUser: {},
      turnipData: {},
      options: {
        chart: {
          zoom: { enabled: false },
          animations: { enabled: false },
        },
        title: {
          test: "Turn-up for Turnips",
          align: "left"
        }
      },
      series: []
    };
  },
  mounted() {
    this.getUserInfo();
    this.getPriceData();
  },
  methods: {
    getPriceData: function() {
      this.axios.get("https://turniprofit-864ba.firebaseio.com/prices.json")
        .then(response => {
          console.log(response.data);
          let days = [];
          for(let day in response.data) {
            let obj = response.data[day];
            obj["date"] = day;
            days.push(obj);
          }
          
          days.sort((a, b) => {
            if (a.date > b.date) return 1;
            else return -1;
          });
          
          let xdays = [];
          console.log("Days sorted: " + days);
          for (let index in days) {
            //console.log(days[index])
            let dayObj = days[index];
            if ("morning" in dayObj) {
              const mornTime = dayObj.date + "M";
              xdays.push(mornTime);
              for (let userId in this.idToUser) {
                console.log(userId)
                if (userId in dayObj.morning) {
                  this.turnipData[userId].push(parseInt(dayObj.morning[userId].price));
                } else {
                  this.turnipData[userId].push(null);
                }
              }
            }
            if ("evening" in dayObj) {
              const eveTime = dayObj.date + "E";
              xdays.push(eveTime);
              for (let userId in this.idToUser) {
                console.log(userId)
                if (userId in dayObj.evening) {
                  this.turnipData[userId].push(parseInt(dayObj.evening[userId].price));
                } else {
                  this.turnipData[userId].push(null);
                }
              }
            }
          }
          this.options = {
            ...this.options,
            ...{ xaxis: { categories: xdays } }
          };
          for (let userId in this.turnipData) {
            this.series.push({
              name: this.idToUser[userId],
              data: this.turnipData[userId]
            });
          }
        });
    },
    getUserInfo: function() {
      this.axios.get("https://turniprofit-864ba.firebaseio.com/users.json")
        .then(response => {
          //console.log(response.data);
          for(let userId in response.data) {
            //console.log("UserId: " + userId + " and name: " + response.data[userId].name);
            this.idToUser[userId] = response.data[userId].name;
            this.turnipData[userId] = [];
          }
        });
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
