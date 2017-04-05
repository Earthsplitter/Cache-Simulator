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
        <p v-if="source === 'file'">{{loadMessage}}</p>
      </section>
      <section>
        <div class="control-panel" v-if="source === 'file'">
          <button>Single Step</button>
          <button>Execute Automatically</button>
        </div>
        <div v-if="source === 'input'" class="control-panel">
          <my-select :model="accessType"></my-select>
        </div>
      </section>
    </aside>
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
        cacheType: 'unionCache',
        source: 'file',
        loadMessage: '',
        unionCacheSize: {
          name: '统一Cache大小',
          options: ['2KB', '4KB', '8KB', '16KB', '32KB', '64KB', '128KB', '256KB', '512KB', '1MB'],
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
          name: ''
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
          }
        }
      }
    }
  }
</script>

<style>
  #app {
    box-sizing: content-box;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    margin-top: 60px;
  }

  aside {
    margin: 0 20px;
    width: 35%;
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
    margin: 20px 0;
    border: solid 2px #00bc9b;
    padding: 30px;
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
</style>
