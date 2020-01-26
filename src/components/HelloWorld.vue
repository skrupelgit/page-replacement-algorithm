<template>
  <div class="container">
  
    <div class="row">
      <div class="col-6">
        <div class="row">
          <div class="col-12">
            <label>
              Introduzca el número de marcos
              <input
                type="number"
                class="form-control"
                v-model.number="nFrames"
              />
            </label>
          </div>
          <div class="col-12">
            <label class>
              Introduzca las páginas separadas por comas
              <input
                type="text"
                class="form-control"
                v-model="rawPageList"
              />
            </label>
          </div>
          <div class="col-12">
            <label>
              Fifo

            <input checked  class="form-control" type="radio" name="method" value="fifo">
            </label>
            <label>LRU

            <input disabled class="form-control" type="radio" name="method" value="fifo">
            </label>
            <label>Optimo

            <input disabled class="form-control" type="radio" name="method" value="fifo">
            </label>
            <label>Reloj
            <input disabled class="form-control" type="radio" name="method" value="fifo">
            </label>
          </div>
        </div>
        <button class="btn btn-primary" @click="calculate">Calcular</button>
      </div>
      <div v-if="pageList.length" class="col-6">
        <h4>Secuencia de páginas</h4>
        <span
          v-for="(page,index) in pageList "
          :key="index"
          :style="{backgroundColor:getPageColor(page)}"
          class="badge"
        >{{page}}</span>
        <h4>Listado de Marcos</h4>
        <span
          v-for="(frame,index) in new Array(nFrames)"
          :key="index"
          class="badge badge-dark"
        >{{index}}</span>
      </div>
    </div>
    <hr />

    <div v-if="results.resultTable.length" class="card">
      <div class="card-body">
        <table class>
          <tr v-for="(frame,index) in new Array(nFrames)" :key="'fr'+index">
            <td>
              <span class="badge badge-dark">{{index}}</span>
            </td>
            <td v-for="(result, resultIndex) in results.resultTable" :key="'tb'+resultIndex">
              <span
                :style="{backgroundColor:getPageColor(result[index])}"
                class="badge"
              >{{result[index]}}</span>
            </td>
          </tr>
        </table>
        <code>
        Fallos de página : {{results.pageFaults}} <br>
        Reemplazos : {{results.replacement}}
        </code>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {},
  data() {
    return {
      nFrames: null,
      rawPageList: null,
      results: {
        resultTable: [],
        pageFaults: 0
      },
      pageList: [],
      pageDescription: []
    };
  },
  methods: {
    generatePageList() {
      /* eslint-disable no-console */

      this.pageDescription = [];
      this.pageList = [];
      if (this.rawPageList) {
        this.pageList = this.rawPageList.split(",");
        for (let page in this.pageList) {
          this.pageDescription[this.pageList[page]] = {
            value: this.pageList[page],
            color: this.randomColor()
          };
        }
      }
    },
    randomColor() {
      let letters = "0123456789ABCDEF";
      let color = "#F";
      for (var i = 0; i < 5; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }

      return color;
    },
    calculate() {
      this.generatePageList();
      let frames = [];
      let replacement=0;
      this.results.resultTable = [];
      let pagesAdded = [];
      for (let frame = 0; frame < this.nFrames; frame++) {
        frames.push(null);
      }

      for (let page in this.pageList) {
        let currentPage = this.pageList[page];
        if (!this.inFrame(currentPage, frames)) {
          if (this.emptyFrames(frames)) {
            for (let frame = 0; frame < this.nFrames; frame++) {
              if (frames[frame] === null) {
                frames[frame] = currentPage;
                pagesAdded.push(currentPage);
                break;
              }
            }
          } else {
            frames[this.getIndexFirstCame(frames, pagesAdded)] = currentPage;
            replacement++;
            pagesAdded.push(currentPage);
          }
        }
        this.results.resultTable.push([...frames]);
      }
      this.results.pageFaults = pagesAdded.length;
      this.results.replacement=replacement;
      this.$forceUpdate();
    },
    emptyFrames(frames) {
      for (let frame = 0; frame < this.nFrames; frame++) {
        if (frames[frame] === null) return true;
      }
      return false;
    },
    inFrame(page, frames) {
      for (let frame = 0; frame < this.nFrames; frame++) {
        if (frames[frame] === page) return true;
      }
      return false;
    },
    getIndexFirstCame(frames, pagesAdded) {
      let indexList = [];
      for (let frame in frames) {
        indexList.push(pagesAdded.lastIndexOf(frames[frame]));
      }
      return frames.lastIndexOf(pagesAdded[Math.min(...indexList)]);
    },
    getPageColor(page) {
      if (typeof this.pageDescription[page] === "undefined") return "#FFFFF";
      else return this.pageDescription[page].color;
    }
  }
};
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
.badge{
  margin:1px;
}
</style>
