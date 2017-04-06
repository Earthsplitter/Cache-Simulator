<template>
  <div id="app">
    <aside>
      <section id="settings">
        <div id="cache-type">
          <input type="radio" id="unionCache" value="unionCache" v-model="cacheType">
          <label for="unionCache">统一Cache</label>
          <input type="radio" id="independentCache" value="independentCache" v-model="cacheType">
          <label for="independentCache">独立Cache</label>
          <my-select v-if="cacheType === 'unionCache'" :model="unionCacheSize"></my-select>
          <div v-else>
            <my-select :model="dataCacheSize"></my-select>
            <my-select :model="instructionCacheSize"></my-select>
          </div>
        </div>
        <my-select :model="blockSize"></my-select>
        <my-select :model="association"></my-select>
        <my-select :model="replacePolicy"></my-select>
        <my-select :model="preFetchPolicy"></my-select>
        <my-select :model="writePolicy"></my-select>
        <my-select :model="writeMissPolicy"></my-select>
        <p>访问地址:</p>
        <input type="radio" id="file" value="file" v-model="source">
        <label for="file">文件</label>
        <input type="radio" id="input" value="input" v-model="source">
        <label for="input">手动输入</label>
        <input v-if="source === 'file'" type="file" @change="fileUpload"/>
        <p id="load-message" v-if="source === 'file'">{{loadMessage}}</p>
      </section>
      <section>
        <div class="control-panel" v-if="source === 'file'">
          <button @click="run">Single Step</button>
          <button @click="executeAll">Execute Automatically</button>
        </div>
        <div v-if="source === 'input'" class="control-panel">
          <my-select :model="accessType"></my-select>
          <label style="margin: 10px" for="address">地址:</label>
          <input v-model="address" id="address" type="text"/>
          <button id="accessButton">Access</button>
        </div>
      </section>
    </aside>
    <main>
      <h1 style="width: 100%;text-align: center">模拟结果</h1>
      <span class="showing-items" v-for="(data,key) in simulationComputed.summary">
        {{key}}: {{data}}
      </span>
      <p style="width: 100%">其中:</p>
      <span class="showing-items" v-for="(data,key) in simulationData.readInstruction">
        {{key}}: {{data}}
      </span>
      <span class="showing-items" v-for="(data,key) in simulationComputed.readInstruction">
        {{key}}: {{data}}%
      </span>
      <span class="showing-items" v-for="(data,key) in simulationData.readData">
        {{key}}: {{data}}
      </span>
      <span class="showing-items" v-for="(data,key) in simulationComputed.readData">
        {{key}}: {{data}}%
      </span>
      <span class="showing-items" v-for="(data,key) in simulationData.writeData">
        {{key}}: {{data}}
      </span>
      <span class="showing-items" v-for="(data,key) in simulationComputed.writeData">
        {{key}}: {{data}}%
      </span>
    </main>
  </div>
</template>

<script>
  import Select from './components/Select.vue'
  import MySelect from "./components/Select";

  export default {
    name: 'app',
    components: {
      MySelect, 'my-select': Select
    },
    data () {
      return {
        unionCache: [],
        unionTime: [],
        loadInstruction: [],
        simulationData: {
          readInstruction: {
            '读指令次数': 0,
            '不命中次数': 0
          },
          readData: {
            '读数据次数': 0,
            '不命中次数': 0,
          },
          writeData: {
            '写数据次数': 0,
            '不命中次数': 0
          }
        },
        cacheType: 'unionCache',
        source: 'file',
        loadMessage: '',
        address: '',
        unionCacheSize: {
          name: '统一Cache大小',
          options: ['2KB', '4KB', '8KB', '16KB', '32KB', '64KB', '128KB', '256KB', '512KB', '1024KB'],
          selected: '64KB'
        },
        dataCacheSize: {
          name: '数据Cache大小',
          options: ['1KB', '2KB', '4KB', '8KB', '16KB', '32KB', '64KB', '128KB', '256KB', '512KB'],
          selected: '32KB'
        },
        instructionCacheSize: {
          name: '指令Cache大小',
          options: ['1KB', '2KB', '4KB', '8KB', '16KB', '32KB', '64KB', '128KB', '256KB', '512KB'],
          selected: '32KB'
        },
        blockSize: {
          name: '块大小',
          options: ['16B', '32B', '64B', '128B', '256B'],
          selected: '32B'
        },
        association: {
          name: '相联度',
          options: ['直接映像', '2路', '4路', '8路', '16路', '32路'],
          selected: '直接映像'
        },
        replacePolicy: {
          name: '替换策略',
          options: ['LRU', 'FIFO', 'RAND'],
          selected: 'LRU'
        },
        preFetchPolicy: {
          name: '预取策略',
          options: ['不预取', '不命中预取'],
          selected: '不预取'
        },
        writePolicy: {
          name: '写策略',
          options: ['写回法', '写直达法'],
          selected: '写回法'
        },
        writeMissPolicy: {
          name: '写不命中的调块策略',
          options: ['按写分配', '不按写分配'],
          selected: '按写分配'
        },
        accessType: {
          name: '访问类型',
          options: ['读指令', '读数据', '写数据'],
          selected: '读指令'
        }
      }
    },
    computed: {
      simulationComputed () {
        return {
          summary: {
            '访问总次数': this.simulationData.readInstruction['读指令次数'] + this.simulationData.readData['读数据次数'] + this.simulationData.writeData['写数据次数'],
            '不命中次数': this.simulationData.readInstruction['不命中次数'] + this.simulationData.readData['不命中次数'] + this.simulationData.writeData['不命中次数'],
            '未命中率': (100 * (this.simulationData.readInstruction['不命中次数'] + this.simulationData.readData['不命中次数'] + this.simulationData.writeData['不命中次数']) / (this.simulationData.readInstruction['读指令次数'] + this.simulationData.readData['读数据次数'] + this.simulationData.writeData['写数据次数'])).toFixed(2) + '%'
          },
          readInstruction: {
            '未命中率': (100 * this.simulationData.readInstruction['不命中次数'] / this.simulationData.readInstruction['读指令次数']).toFixed(2)
          },
          readData: {
            '未命中率': (100 * this.simulationData.readData['不命中次数'] / this.simulationData.readData['读数据次数']).toFixed(2)
          },
          writeData: {
            '未命中率': (100 * this.simulationData.writeData['不命中次数'] / this.simulationData.writeData['写数据次数']).toFixed(2)
          }
        }
      }
    },
    methods: {
      fileUpload (e) {
        let file = e.target.files[0]
        if (file.name.match(/.din$/) === null) {
          this.loadMessage = 'Wrong file type. Only accept *.din'
        } else {
          this.loadMessage = 'Loading'
          let fileReader = new FileReader()
          fileReader.readAsText(file)
          fileReader.onload = () => {
            this.loadMessage = 'load Success'
            this.loadInstruction = fileReader.result.split(/\s/)
            this.unionCache = []
            this.unionTime = []
            this.simulationData.writeData['写数据次数'] = 0
            this.simulationData.readData['读数据次数'] = 0
            this.simulationData.readInstruction['读指令次数'] = 0
            this.simulationData.writeData['不命中次数'] = 0
            this.simulationData.readData['不命中次数'] = 0
            this.simulationData.readInstruction['不命中次数'] = 0
          }
        }
      },
      run () {
        let type = this.loadInstruction[2*this.simulationComputed.summary['访问总次数']]
        let address = this.loadInstruction[2*this.simulationComputed.summary['访问总次数']+1]
        switch (type) {
          case '0':
            this.simulationData.readData['读数据次数'] ++
            break;
          case '1':
            this.simulationData.writeData['写数据次数'] ++
            break;
          case '2':
            this.simulationData.readInstruction['读指令次数'] ++
            break;
        }
        if (type === '2' && this.cacheType === 'independentCache') {
          this.fetchInstruction()
        } else {
          this.accessData(type, address)
        }
      },
      executeAll () {
        console.log(this.loadInstruction)
        while (this.simulationComputed.summary['访问总次数']*2 < this.loadInstruction.length) {
          this.run()
        }
      },
      fetchInstruction () {

      },
      accessData (type, address) {
        let realAddress = Math.floor(parseInt(address, 16) / this.blockSize.selected.slice(0, -1))

        if (this.cacheType === 'unionCache') {
          let miss = true
          let cacheBlockNumber = this.unionCacheSize.selected.slice(0, -2) * 1024 / this.blockSize.selected.slice(0, -1)
          let setNumber = cacheBlockNumber / ( (this.association.selected === '直接映像') ? 1 : this.association.selected.slice(0, -1) )
          let cacheAddress = realAddress % setNumber
          for (let i = cacheAddress; i < cacheBlockNumber; i += setNumber) {
            if (this.unionCache[i] === realAddress) {
              miss = false
              if (this.replacePolicy.selected === 'LRU') {
                this.unionTime[i] = this.simulationComputed.summary['访问总次数'] - 1
              }
            }
          }
          if (miss === true) {
            let replaceAddress = cacheAddress
            for (let i = replaceAddress + setNumber; i < cacheBlockNumber; i += setNumber) {
              if (this.unionTime[i] < this.unionTime[replaceAddress]) {
                replaceAddress = i
              }
            }
            this.unionCache[replaceAddress] = realAddress
            this.unionTime[replaceAddress] = this.simulationComputed.summary['访问总次数'] - 1
            switch (type) {
              case '0':
                this.simulationData.readData['不命中次数'] ++
                break;
              case '1':
                this.simulationData.writeData['不命中次数'] ++
                break;
              case '2':
                this.simulationData.readInstruction['不命中次数'] ++
                break;
            }
          }
        } else {

        }
      }
    }
  }
</script>

<style>
  #app {
    display: flex;
    box-sizing: content-box;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    margin-top: 60px;
  }

  aside {
    margin: 0 20px;
    width: 65%;
  }

  main {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    border: solid 2px #00bc9b;
    font-size: 18px;
    padding: 30px;
  }

  .showing-items {
    display: block;
    text-align: center;
    width: 33%;
  }

  #settings {
    border: solid 2px #00bc9b;
    font-size: 18px;
    padding: 30px;
  }

  #cache-type {
    margin: 20px 0;
    border-bottom: dotted 1px black;
  }

  input[type="radio"] {
    margin-left: 10px;
  }

  .control-panel {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    margin: 20px 0;
    border: solid 2px #00bc9b;
    padding: 20px;
  }

  button {
    background-color: white;
    border: solid 1px #00bc9b;
    margin: 0 20px;
    height: 45px;
    width: 120px;
    border-radius: 15px;
    cursor: pointer;
  }

  button:active {
    color: white;
    background-color: #00bc9b;
  }

  button:focus {
    outline: none;
  }

  input[type='text'] {
    height: 20px;
    margin: 10px;
  }
  #load-message {
    text-align: center;
    margin-bottom: -10px;
    font-weight: bold;
    color: red;
  }
</style>
