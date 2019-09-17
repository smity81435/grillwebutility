<template>
  <div class="chart">
    <div class="chartbox">
      <h1>{{deviceName || "Searching for Grill..."}}</h1>
      <ul>
        <li class="availableProbe">{{activeSensor}}</li>
      </ul>
      <v-chart class="lines" :options="line" />
    </div>

  </div>
</template>

<script>
import ECharts from 'vue-echarts'
import 'echarts/lib/chart/line'
import 'echarts/lib/component/tooltip'
import 'echarts/lib/chart/gauge'
var kettledata=[];
var probe1data=[];
var probe2data=[];
var times=[];

export default {
  name: 'chart',
  components:{
    'v-chart': ECharts
  },
  props:{
    activeSensor:{
      type: String,
      requried: true,
    },
    deviceName:{
      type: String,
      required: false,
    },
    kettle:{
      type: Array,
      required: true,
    },
    probeone:{
      type: Array,
      required: true,
    },
    probetwo:{
      type: Array,
      required: true,
    },
    logging:{
      type: Boolean,
      required: true,
    }
  },
  methods:{
    randomData(){
      let d = new Date();
      let now = d.toTimeString().split(' ')[0]
      let value = Math.random()*(1000-300)+300;
      var plot=[now, Math.round(value)]
      return {
        plot
      }
    },

  },
  data(){
    return{
      gauge:{

      },
      line: {
        textStyle:{
          color: "#fff8ed",
        },
        color:['#2ECC71','#8E44AD','#3498DB'],
        title: {
          text: 'Grill Diagnostic Data'
        },
        legend: {
          data: [{
            name: "Kettle",
            icon: 'circle',
          },{
            name:'Probe 1',
            icon: 'circle',
          },{
            name: 'Probe 2',
            icon: 'circle'
          }]
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          }
        },
        xAxis:{
          type: "time",
          splitLine:{
            show:false
          },
          data: times,
        },
        yAxis: {
          type: "value",
          title: "Temperature in °F"
        },
        series: [
            {
              name: "Kettle Temp",
              type: "line",
              data: this.kettle,
            },
            {
              name: "Probe 1",
              type: "line",
              data: this.probeone,
              
            },
            {
              name: "Probe 2",
              type: "line",
              data: this.probetwo,
            }
        ],
        animationDuration: 1000
      }

    }
  },
  mounted(){
    
    
    // setInterval(()=>{
    //     if(this.logging){
    //       let d = new Date()
    //       let randints = []
    //       for(var i = 0; i <3; i ++){
    //         let val = Math.random()*(700)+300;
    //         randints.push(Math.floor(val))
    //       }
    //       let time = d.toTimeString().split(' ')[0]
    //       kettledata.push([d,randints[0]]);
    //       probe1data.push([d,randints[1]]);
    //       probe2data.push([d,randints[2]]);
    //     }
    //  } ,1000);
      

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.chart{
  border: 1px solid #888888;
  height: 100%;
  width: 100%;
}
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
.echarts{
  width: 100%;
  height: 90%;
}
.chartbox{
  background: #1e1e1e;
  color: #fff8ed;
  height: 80%;
  width: 90%;
}
.lines{
  width: 100% !important;
  margin: 0 auto !important;
}
</style>
