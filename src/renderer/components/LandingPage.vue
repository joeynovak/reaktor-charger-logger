<template>
  <div id="wrapper">
    <img id="logo" src="~@/assets/logo.png" alt="reaktor-charger">
    <div class="title">
      Welcome to the Reaktor Charger Logger
    </div>
    <main>
      <div class="left-side">
        Batteries and Session tree will go here...
      </div>

      <div class="right-side">
        <select v-model="selectedPort" >
          <option v-for="serialPort in serialPorts" :value="serialPort.comName">{{ serialPort.comName }}</option>
        </select>

        <div>Serial Port</div>
        <span>Baud Rate (Usually 9600)</span>
        <select v-model="baudRate">
          <option v-for="rate in [ '9600', '19200', '28800', '56700', '115200']" v-model="baudRate">{{ rate }}</option>
        </select>

        <button @click="exit">Quit</button>
        <button v-show="!portOpen" @click="openPort">Open Port</button>
        <button v-show="portOpen" @click="closePort">Close Port</button>
        <button @click="refreshPorts">Refresh Port List</button>
        <div v-show="chargerData.mode == 1">
          Charging
        </div>
        <div v-show="chargerData.mode == 6">
          Done
        </div>
        <table>
          <tr>
            <th>Stat</th>
            <th>Value</th>
          </tr>
          <tr>
            <td>Mode</td><td>{{ chargerData.mode }}</td>
          </tr>
          <tr>
            <td>Input Voltage</td><td>{{ chargerData.inputVoltage }} V</td>
          </tr>
          <tr>
            <td>Battery Voltage</td><td>{{ chargerData.batteryVoltage }} V</td>
          </tr>
          <tr>
            <td>Charge Current</td><td>{{ chargerData.chargeCurrent | formatNumberTwoDecimals }} A</td>
          </tr>
          <tr>
            <td>Energy Placed in Battery</td><td>{{ chargerData.chargedMah / 1000 }} Ah</td>
          </tr>
          <tr>
            <td>Cell 1 Voltage</td><td>{{ chargerData.cellVoltage[0] }} V</td>
          </tr>
          <tr>
            <td>Cell 2 Voltage</td><td>{{ chargerData.cellVoltage[1] }} V</td>
          </tr>
          <tr>
            <td>Cell 3 Voltage</td><td>{{ chargerData.cellVoltage[2] }} V</td>
          </tr>
          <tr>
            <td>Cell 4 Voltage</td><td>{{ chargerData.cellVoltage[3] }} V</td>
          </tr>
          <tr>
            <td>Cell 5 Voltage</td><td>{{ chargerData.cellVoltage[4] }} V</td>
          </tr>
          <tr>
            <td>Cell 6 Voltage</td><td>{{ chargerData.cellVoltage[5] }} V</td>
          </tr>
        </table>
      </div>
    </main>
  </div>
</template>

<script>
  import SystemInformation from './LandingPage/SystemInformation'
  import SerialPort from 'serialport'
  import Readline from '@serialport/parser-readline'

  const remote = require('electron').remote

export default {
    name: 'landing-page',
    components: { SystemInformation },
    data: () => {
      return {
        serialPorts: [],
        portOpen: false,
        selectedPort: null,
        baudRate: '9600',
        currentPort: null,
        currentParser: null,
        chargerData: {
          cellVoltage: [],
          batteryVoltage: 0,
          inputVoltage: 0,
          mode: '',
          mode2: '',
          chargedMah: '',
          chargeCurrent: ''
        }
      }
    },
    methods: {
      exit () {
        if (this.portOpen) {
          this.closePort()
        }
        var window = remote.getCurrentWindow()
        window.close()
      },
      init () {

      },
      refreshPorts () {
        var vm = this
        SerialPort.list((err, ports) => {
          if (err) {
            console.log(err)
          } else {
            console.log(ports)
            vm.serialPorts = ports
          }
        })
      },
      parseData (data) {
        var parsedData = {cellVoltage: []}
        var splitData = data.split(';')
        // This is the start characters
        // parsedData.mode = splitData[0]
        parsedData.mode = splitData[1]
        parsedData.inputVoltage = splitData[3] / 1000
        parsedData.batteryVoltage = splitData[4] / 1000
        // charge Current is in centiAmps minstead of ma
        parsedData.chargeCurrent = splitData[5] / 100
        parsedData.chargedMah = splitData[14]
        parsedData.cellVoltage[0] = splitData[6] / 1000
        parsedData.cellVoltage[1] = splitData[7] / 1000
        parsedData.cellVoltage[2] = splitData[8] / 1000
        parsedData.cellVoltage[3] = splitData[9] / 1000
        parsedData.cellVoltage[4] = splitData[10] / 1000
        parsedData.cellVoltage[5] = splitData[11] / 1000

        return parsedData
      },
      closePort () {
        if (this.currentPort != null) {
          this.currentPort.close()
        }
        this.portOpen = false
      },
      openPort () {
        var vm = this
        this.currentPort = new SerialPort(this.selectedPort)
        this.currentParser = this.currentPort.pipe(new Readline({ delimiter: '\r\n' }))
        this.currentParser.on('data', function (data) { vm.updateChargerData(vm.parseData(data)); console.log(data) })
        this.portOpen = true
      },
      updateChargerData (data) {
        this.chargerData.inputVoltage = data.inputVoltage
        this.chargerData.batteryVoltage = data.batteryVoltage
        this.chargerData.chargeCurrent = data.chargeCurrent
        this.chargerData.chargedMah = data.chargedMah
        this.chargerData.mode = data.mode

        this.chargerData.cellVoltage[0] = data.cellVoltage[0]
        this.chargerData.cellVoltage[1] = data.cellVoltage[1]
        this.chargerData.cellVoltage[2] = data.cellVoltage[2]
        this.chargerData.cellVoltage[3] = data.cellVoltage[3]
        this.chargerData.cellVoltage[4] = data.cellVoltage[4]
        this.chargerData.cellVoltage[5] = data.cellVoltage[5]
      }
    },
    computed: {
    },
    watch: {
      selectedPort () {
        if (this.currentPort != null) {
          this.currentParser.closePort()
          this.currentPort = null
          this.currentParser = null
        }
      },
      baudRate () {
        if (this.currentPort != null) {

        }
      }
    },
    mounted () {
      this.refreshPorts()
    },
    filters: {
      formatNumberTwoDecimals (value) {
        return value === undefined ? 0.00 : parseFloat(value).toFixed(2)
      }

    }
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body { font-family: 'Source Sans Pro', sans-serif; }

  #wrapper {
    background:
      radial-gradient(
        ellipse at top left,
        rgba(255, 255, 255, 1) 40%,
        rgba(229, 229, 229, .9) 100%
      );
    height: 100vh;
    padding: 60px 80px;
    width: 100vw;
  }

  #logo {
    height: auto;
    margin-bottom: 20px;
    width: 312px;
  }

  main {
    display: flex;
    justify-content: space-between;
  }

  main > div { flex-basis: 50%; }

  .left-side {
    display: flex;
    flex-direction: column;
    flex-basis: 30%;
  }

  .welcome {
    color: #555;
    font-size: 23px;
    margin-bottom: 10px;
  }

  .title {
    color: #2c3e50;
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 6px;
  }

  .title.alt {
    font-size: 18px;
    margin-bottom: 10px;
  }

  .doc p {
    color: black;
    margin-bottom: 10px;
  }

  .doc button {
    font-size: .8em;
    cursor: pointer;
    outline: none;
    padding: 0.75em 2em;
    border-radius: 2em;
    display: inline-block;
    color: #fff;
    background-color: #4fc08d;
    transition: all 0.15s ease;
    box-sizing: border-box;
    border: 1px solid #4fc08d;
  }

  .doc button.alt {
    color: #42b983;
    background-color: transparent;
  }
</style>
