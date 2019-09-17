<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <div class="controls">
      <div class="bleButton" @click="connect()">Search for Nearby Grills</div>
      <ul class="detailList">
        <h2>{{"Temperature Sensors"}}</h2>
        <li class="grill" v-for="(item, index) in grills" :key="index">{{item}}: Connected</li>
        
        <div class="gauges">
          <div class="gaugeBox">
            <v-chart class="gauge" :options="gauge1"></v-chart>
            <h3 class="khead">Kettle Temp °F</h3>
          </div>
          <div class="gaugeBox">
            <v-chart class="gauge" :options="gauge2"></v-chart>
            <h3 class="p1head">Probe 1 Temp °F</h3>
          </div>
          <div class="gaugeBox">
            <v-chart class="gauge" :options="gauge3"></v-chart>
            <h3 class="p2head">Probe 2 Temp °F</h3>
          </div>
        </div>
      </ul>
    </div>
    <div class="visarea">
      <chart :activeSensor="activeSensor"
        :data="grilldata"
        :kettle="kettle"
        :probeone="probeone"
        :probetwo="probetwo"
        :stokefan="stokefan"
        :coolfan="coolfan"
        :deviceName="deviceName" 
        :logging="logging" />
      <div class="chartControls">
        <div class="controlbutton" @click="showTemps">Temperature</div>
        <div class="controlbutton" @click="showSpeeds">Fan Speeds</div>
        <div class="controlbutton" @click="toggleLogs">{{this.loggingbutton}}</div>
      </div>
    </div>
    <div class="fangauges">
      <h2>Fan Speeds</h2>
      <div class="gaugeBox">
        <v-chart :options="gauge4" class="gauge"></v-chart>
        <h3 class="sfanhead">Stoke Fan Speed RPM</h3>
      </div>
      <div class="gaugeBox">
        <v-chart :options="gauge5" class="gauge"></v-chart>
        <h3 class="cfanhead"></h3>
        <h3 class="sfanhead">Cool Fan Speed RPM</h3>
      </div>
      <div class="gaugeBox"></div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
// characteristics
// 002 = Kettle
// 003 =  Probe 1
// 004 = Probe 2
// 100 = Stoke Fan Speed
// 101 = Cool Fan Speed
import ECharts from 'vue-echarts'
import 'echarts/lib/chart/gauge'
import chart from '@/components/chart.vue'
var kettleGauge=0;
var p1Gauge = 0;
var p2Gauge = 0;
var sfGauge=0;
var cfGauge=0;
export default {
  name: 'home',
  components: {
    chart,
    'v-chart': ECharts
  },
  data(){
    return{
      tg1:0,
      tg2:0,
      tg3:0,
      activedevice:{},
      loggingbutton:"Pause Logging",
      activeSensor: "Temperature Sensors",
      logging: true,
      time: new Date(),
      grilldata:[],
      grills:[],
      deviceName:"",
      device:{},
      service:{},
      characteristics:[
        "00000002-1212-efde-1523-785feabcd123",
        "00000003-1212-efde-1523-785feabcd123",
        "00000004-1212-efde-1523-785feabcd123",
        "00000100-1212-efde-1523-785feabcd123",
        "00000101-1212-efde-1523-785feabcd123",
        ],
      kettle:[],
      probeone:[],
      probetwo:[],
      stokefan:[],
      coolfan:[],
      kettleTemp:0,
      p1Temp: 0,
      p2Temp: 0,
      sFanSpeed: 0,
      cFanSpeed: 0,
      gauge1:{
        series:[
          {
            title: "Kettle Temp",
            name:"Kettle",
            type: "gauge",
            min: 0,
            max: 1000,
            splitNumber: 4,
            radius: '95%',
            data: [{value: kettleGauge, name: "°F"}],
            axisLine: {            // 坐标轴线
                lineStyle: {       // 属性lineStyle控制线条样式
                    color: [[0.09, 'skyblue'],[.5,'#F1C40F'],[0.82, '#E67E22'],[1, '#ff4500']],
                    width: 3,
                    shadowColor : '#fff', //默认透明
                    shadowBlur: 10
                }
            },
            splitLine: {           // 分隔线
                length :5,         // 属性length控制线长
                lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                    width:2,
                    color: '#fff',
                    shadowColor : '#fff', //默认透明
                    shadowBlur: 10
                }
            },
            pointer: {
                width:3,
                shadowColor : '#fff', //默认透明
                shadowBlur: 5
            },
          },
        ]
      },
      gauge2:{
        series:[
          {
            name:"Probe 1",
            type: "gauge",
            min: 0,
            max: 1000,
            splitNumber: 4,
            radius: '95%',
            data: [{value: p1Gauge,namne: "°F"}],
            axisLine: {            // 坐标轴线
              lineStyle: {       // 属性lineStyle控制线条样式
                  color: [[0.09, 'skyblue'],[.5,'#F1C40F'],[0.82, '#E67E22'],[1, '#ff4500']],
                  width: 3,
                  shadowColor : '#fff', //默认透明
                  shadowBlur: 10
              }
            },
            splitLine: {           // 分隔线
                length :5,         // 属性length控制线长
                lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                    width:2,
                    color: '#fff',
                    shadowColor : '#fff', //默认透明
                    shadowBlur: 10
                }
            },
            pointer: {
                width:3,
                shadowColor : '#fff', //默认透明
                shadowBlur: 5
            },
          },
        ]
      },
      gauge3:{
        color: "white",
        height: 200,
        series:[
          {
            name:"Probe 2",
            type: "gauge",
            min: 0,
            max: 1000,
            splitNumber: 4,
            radius: '95%',
            data: [{value: p2Gauge,name:"°F"}],
            axisLine: {            // 坐标轴线
                lineStyle: {       // 属性lineStyle控制线条样式
                    color: [[0.09, 'skyblue'],[.5,'#F1C40F'],[0.82, '#E67E22'],[1, '#ff4500']],
                    width: 3,
                    shadowColor : '#fff', //默认透明
                    shadowBlur: 10
                }
            },
            splitLine: {           // 分隔线
              length :5,         // 属性length控制线长
              lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                  width:2,
                  color: '#fff',
                  shadowColor : '#fff', //默认透明
                  shadowBlur: 10
              }
            },
            pointer: {
                width:3,
                shadowColor : '#fff8ed', //默认透明
                shadowBlur: 5
            },
          }
        ]
      },
      gauge4:{
        color: "white",
        height: 200,
        series:[
          {
            name:"Probe 2",
            type: "gauge",
            min: 0,
            max: 10000,
            splitNumber: 4,
            radius: '95%',
            data: [{value: sfGauge,name:"°F"}],
            axisLine: {            // 坐标轴线
                lineStyle: {       // 属性lineStyle控制线条样式
                    color: [[0.09, 'skyblue'],[.5,'#F1C40F'],[0.82, '#E67E22'],[1, '#ff4500']],
                    width: 3,
                    shadowColor : '#fff', //默认透明
                    shadowBlur: 10
                }
            },
            splitLine: {           // 分隔线
              length :5,         // 属性length控制线长
              lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                  width:2,
                  color: '#fff',
                  shadowColor : '#fff', //默认透明
                  shadowBlur: 10
              }
            },
            pointer: {
                width:3,
                shadowColor : '#fff8ed', //默认透明
                shadowBlur: 5
            },
          }
        ]
      },
      gauge5:{
        color: "white",
        height: 200,
        series:[
          {
            name:"Probe 2",
            type: "gauge",
            min: 0,
            max: 10000,
            splitNumber: 4,
            radius: '95%',
            data: [{value: cfGauge,name:"°F"}],
            axisLine: {            // 坐标轴线
                lineStyle: {       // 属性lineStyle控制线条样式
                    color: [[0.09, 'skyblue'],[.5,'#F1C40F'],[0.82, '#E67E22'],[1, '#ff4500']],
                    width: 3,
                    shadowColor : '#fff', //默认透明
                    shadowBlur: 10
                }
            },
            splitLine: {           // 分隔线
              length :5,         // 属性length控制线长
              lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                  width:2,
                  color: '#fff',
                  shadowColor : '#fff', //默认透明
                  shadowBlur: 10
              }
            },
            pointer: {
                width:3,
                shadowColor : '#fff8ed', //默认透明
                shadowBlur: 5
            },
          }
        ]
      }
    }
  },
  methods:{
    toggleLogs(){
      this.logging = !this.logging;
    },
    showTemps(){
      this.activeSensor="Temperature Sensors";
    },
    showSpeeds(){
      this.activeSensor="Fan Speeds";
    },
    async dataCycle(){

    },
    handlechange(event){
      let d = new Date();
      if(event.target.uuid == "00000002-1212-efde-1523-785feabcd123"){
        this.handleKettleChanged(event,d);
      }else if(event.target.uuid == "00000003-1212-efde-1523-785feabcd123"){
        this.handleProbeOneChanged(event,d);
      }else if(event.target.uuid == "00000004-1212-efde-1523-785feabcd123"){
        this.handleProbeTwoChanged(event,d);
      }else if(event.target.uuid == "00000100-1212-efde-1523-785feabcd123"){
        this.handleSFChanged(event);
      }else if(event.target.uuid == "00000101-1212-efde-1523-785feabcd123"){
        this.handleCFChanged(event,d);
      }
    },
    handleKettleChanged(event,d){
      var data = event.target.value.getUint8(2);
      // console.log("Kettle Notification: "+data)
      if(data != this.kettleTemp){
        this.kettleTemp = data;
        kettleGauge = this.kettleTemp;
        this.kettle.push([d,data]);
        this.gauge1.series[0].data[0].value=kettleGauge;
      }
    },
    handleProbeOneChanged(event,d){
      var data = event.target.value.getUint8(2);
      // console.log("Probe 1 Notification: "+data)
      if(data != this.p1Temp){
        this.p1Temp = data;
        p1Gauge = this.p1Temp;
        this.probeone.push([d,data]);
        this.gauge2.series[0].data[0].value=p1Gauge;
      }
    },
    handleProbeTwoChanged(event,d){
      var data = event.target.value.getUint8(2);
      // console.log("Probe 2 Notification: "+data)
      if(data != this.p2Temp){
        this.p2Temp = data;
        this.probetwo.push([d,data]);
        p2Gauge = this.p2Temp;
        this.gauge3.series[0].data[0].value=p2Gauge;
      }
    },
    handleSFChanged(event,d){
      var data = event.target.value.getUint8(2);
      if(data != this.sFanSpeed){
        this.sFanSpeed = data;
        this.stokefan.push([d,data]);
        sfGauge = data;
        this.gauge4.series[0].data[0].value = sfGauge;
      }
    },
    handleCFChanged(event,d){
      var data = event.target.value.getUint8(2);
      if(data != this.cFanSpeed){
        this.cFanSpeed = data;
        this.coolfan.push([d,data]);
        cfGauge = data;
        this.gauge5.series[0].data[0].value = cfGauge;
      }
    },
    onDisconnected(event){
      let device = event.target;
      alert('Device '+device.name+ ' has been disconnected.');
    },
    connect(){
      alert("Searching for grills.")
      navigator.bluetooth.requestDevice({
        // filter the devices available
        acceptAllDevices: true,
        optionalServices:["00000001-1212-efde-1523-785feabcd123"]
      })
      .then((device)=>{
        this.activedevice = device;
        this.deviceName = device.name;
        this.grills.push(device.name);
        device.addEventListener('gattserverdisconnected', this.onDisconnected)
        return device.gatt.connect();
      })
      .then((server)=>{
        //SERVICE UUID WILL CHANGE
        return server.getPrimaryService("00000001-1212-efde-1523-785feabcd123");
      })
      .then((service)=>{
        return service.getCharacteristics();
      })
      .then( characteristics => {
        for(var i = 0; i < characteristics.length; i++){
          characteristics[i].startNotifications();
          characteristics[i].addEventListener('characteristicvaluechanged',this.handlechange)
          console.log(characteristics[i]);
        }
      })
      .catch((error)=>{
        alert(error);
        console.log(error);
      })
    },
  }
}
</script>
<style lang="scss">
.export{
  transition: all 1s;
  
  background: #fff8ed;
  color: #1e1e1e;
  max-width: 30%;
  margin: 10px auto;
  border-radius: 3px;
  &:hover{
    cursor:pointer;
    background: rgb(62, 223, 134);
  }

}
.visarea{
  height: 100%;
}
.controls{
  border: 1px solid #888888;
  display: flex;
  flex-flow: column;
  justify-content: center;
  align-content: center;
}
.chartControls{
  display: flex;
  border: 1px solid #888888;
  justify-content: center;

}
.controlbutton{
  transition: all .25s;
  // border: 1px solid #fff8ed;
  max-width: 20%;
  color: #fff8ed;
  margin: 10px auto;
  padding: 10px 20px;
  border-radius: 4px;
  background: rgb(43, 43, 43);
  font-weight: 600;
  &:hover{
    cursor: pointer;
    color: #1e1e1e;
    background: rgb(62, 223, 134);
  }

}
.bleButton{
  z-index: 9999;
  font-family: "Avenir";
  transition: all 1s;
  width: 80%;
  margin: 10px auto;
  font-size: 20px;
  border-radius: 3px;
  padding: 5px;
  background: rgb(43, 43, 43);
  color: #fff8ed;
  &:hover{
    cursor: pointer !important;
    background: rgb(62, 223, 134);
    color: #1e1e1e;
  }
}
.grill{
  background: limegreen;
  color: black;
  width: 80%;
  margin: 5px auto;
  border-radius: 3px;
}
.detailList{
  list-style: none;
  padding: 0;
  h2{
    color: #fff8ed;
  }
}
.home{
  display: grid;
  grid-template-columns: 1fr 4fr 1fr;
  padding: 20px;
  margin: 0px;
  width: 90%;
  height: 90%;
  z-index: 999;
  background: #1e1e1e;
}
.gauges{
  display: flex;
  justify-content: center;
  flex-flow: column;
    h3{
    color: #fff8ed;
  }
}
.gauge{

    height: 8vw !important;
    width: 10vw !important;
    margin: 0px auto !important;
    padding: 0px !important;
    overflow: visible !important;
}
.gaugeBox{
  background: rgb(43, 43, 43);
  width: 80%;
  padding: 5px;
  margin: 10px auto;
  border-radius: 5px;
}
.khead{
  color: #2ECC71 !important;
}
.p1head{
  color:#8E44AD !important;
}
.p2head{
 color: #3498DB !important;
}
</style>
