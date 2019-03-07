<template>
  <Window title="reaktor-charger" width="800" height="600" margined v-on:close="exit">
    <Box>
      <Text>Serial Port</Text>
      <DropdownList v-bind:items="serialPorts" v-model="selectedPort"/>
      <Text>Baud Rate (Usually 9600)</Text>
      <DropdownList v-bind:items="[ '9600', '19200', '28800', '56700', '115200']" v-model="baudRate"/>
      <Text>{{ data }}
      123
      5678
      </Text>
      <Button v-show="portOpen" @click="exit">Quit</Button>
      <Button v-show="!portOpen" @click="openPort">Open Port</Button>
      <Button v-show="portOpen" @click="closePort">Close Port</Button>
      <Button @click="refreshPorts">Refresh Port List</Button>

    </Box>
  </Window>
</template>

<script>
  import SerialPort from 'serialport';
  import Readline from '@serialport/parser-readline';



export default {
  data: () => { return {
    serialPorts: [],
    serialPortData: ' Testing 123 ',
    portOpen: false,
    selectedPort: null,
    baudRate: null,
    currentPort: null,
    currentParser: null
  }},
  methods: {
    exit() {
      this.closePort();
      this.$exit();
    },
    init(){
      var vm = this;

    },
    refreshPorts(){
      var vm = this;
      SerialPort.list((err, ports)=>{
        console.log(ports);
        vm.serialPorts = ports;
      });
    },
    parseData(data){
      var parsedData = {};


      return parsedData;
    },
    closePort(){
      this.currentParser.closePort();
    },
    openPort(){
      this.currentPort = new SerialPort(this.serialPorts[selectedPort]);
      this.currentParser = port.pipe(new Readline({ delimiter: '\r\n' }));
      parser.on('data', function(data) {vm.parseData(data); console.log(data);})
    }
  },
  computed: {
    data: function () {
      return this.serialPortData;
    }
  },
  watch: {
    selectedPort(){
      if(this.currentPort != null){
        this.currentParser.closePort();
        this.currentPort = null;
        this.currentParser = null;
      }
    },
    baudRate(){
      if(this.currentPort != null){

      }
    }
  }
}


</script>
