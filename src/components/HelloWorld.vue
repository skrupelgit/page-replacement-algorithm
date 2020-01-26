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
                    <div class="col-12 methods">
                        <label>
                            Fifo
                            <input class="form-control" v-model="method" type="radio" name="method" value="fifo"/>
                        </label>
                        <label>
                            LRU
                            <input class="form-control" v-model="method" type="radio" name="method" value="lru"/>
                        </label>
                        <label>
                            Óptimo
                            <input class="form-control" v-model="method" type="radio" name="method" value="opt"/>
                        </label>
                        <label>
                            Reloj
                            <input class="form-control"  v-model="method" type="radio" name="method"
                                   value="clock"/>
                        </label>
                    </div>
                </div>
                <button class="btn btn-primary" @click="calculate">Calcular</button>
            </div>
            <div v-if="pageDescription.length" class="col-6">
                <h4>Listado de páginas</h4>
                <span v-for="(page,index) in pageDescription " :key="'pl'+index">
          <span
                  v-if="page"
                  :style="{backgroundColor:page.color}"
                  class="badge"
          >{{page.value}}</span>
        </span>
                <h4>Secuencia de páginas</h4>
                <span
                        v-for="(page,index) in pageList "
                        :key="'psec'+index"
                        :style="{backgroundColor:getPageColor(page)}"
                        class="badge"
                >{{page}}</span>
                <h4>Listado de Marcos</h4>
                <span
                        v-for="(frame,index) in new Array(nFrames)"
                        :key="'fl'+index"
                        class="badge badge-dark"
                >{{index}}</span>
            </div>
        </div>
        <hr/>

        <div v-if="results.resultTable.length" class="card">
            <div class="card-body">
                <table v-if="resultsMethod!=='clock'" class>
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
                <table v-else class>
                    <tr v-for="(frame,index) in new Array(nFrames)" :key="'fr'+index">
                        <td>
                            <span class="badge badge-dark">{{index}}</span>
                        </td>
                        <td v-for="(result, resultIndex) in results.resultTable" :key="'tb'+resultIndex">
                            <span
                                    :style="{backgroundColor:getPageColor(result.pages[index].value)}"
                                    class="badge">
                                                            <span v-if="result.pointer===index">-></span>

                  {{result.pages[index].value}}<span
                                    v-if="result.pages[index].reference">*</span>
              </span>
                        </td>
                    </tr>
                </table>
                <code>
                    Fallos de página : {{results.pageFaults}}
                    <br/>
                    Reemplazos : {{results.replacement}}
                </code>
            </div>
        </div>
    </div>
</template>

<script>
    import lodashCloneDeep from "lodash.clonedeep"

    export default {
        name: "HelloWorld",
        props: {},
        data() {
            return {
                nFrames: null,
                rawPageList: null,
                method: 'fifo',
                resultsMethod:null,
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
                switch (this.method) {
                    case "fifo":
                        this.calculateFifo();
                        break;
                    case "lru":
                        this.calculateLru();
                        break;
                    case "opt":
                        this.calculateOpt();
                        break;
                    case "clock":
                        this.calculateClock();
                        break;
                    default:
                        break;

                }
                this.resultsMethod=this.method;


            },
            calculateFifo() {
                let frames = [];
                let replacement = 0;
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
                this.results.replacement = replacement;
                this.$forceUpdate();
            },
            calculateLru() {
                let frames = [];
                let replacement = 0;
                this.results.resultTable = [];
                let pagesAdded = [];
                let pagesReferenced = [];
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
                                    pagesReferenced.push(currentPage)
                                    break;
                                }
                            }
                        } else {
                            frames[this.getIndexFirstReferenced(frames, pagesReferenced)] = currentPage;
                            replacement++;
                            pagesAdded.push(currentPage);
                            pagesReferenced.push(currentPage)

                        }
                    } else {
                        pagesReferenced.push(currentPage)
                    }
                    this.results.resultTable.push([...frames]);
                }
                this.results.pageFaults = pagesAdded.length;
                this.results.replacement = replacement;
                this.$forceUpdate();
            },

            calculateOpt() {
                let frames = [];
                let replacement = 0;
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
                            frames[this.getPageReferencedLaterIndex(frames, page)] = currentPage;
                            replacement++;
                            pagesAdded.push(currentPage);

                        }
                    }

                    this.results.resultTable.push([...frames]);
                }
                this.results.pageFaults = pagesAdded.length;
                this.results.replacement = replacement;
                this.$forceUpdate();
            },
            calculateClock() {
                let frames = {pages: [], pointer: 0};
                let replacement = 0;
                this.results.resultTable = [];
                let pagesAdded = [];
                for (let frame = 0; frame < this.nFrames; frame++) {
                    frames.pages.push({value: null, reference: 0});
                }

                for (let page in this.pageList) {
                    let currentPage = this.pageList[page];
                    if (!this.inFrameClock(currentPage, frames)) {
                        if (frames.pages[frames.pointer].value === null) {
                            frames.pages[frames.pointer].value = currentPage;
                            frames.pages[frames.pointer].reference = 1;
                            pagesAdded.push(currentPage);
                            frames.pointer = this.increaseClockPointer(frames.pointer);
                        } else {
                            /* eslint-disable no-constant-condition */
                            while (true) {
                                if (frames.pages[frames.pointer].reference === 1) {
                                    frames.pages[frames.pointer].reference = 0;
                                    frames.pointer = this.increaseClockPointer(frames.pointer);

                                } else {
                                    frames.pages[frames.pointer].value = currentPage;
                                    frames.pages[frames.pointer].reference = 1;
                                    frames.pointer = this.increaseClockPointer(frames.pointer);
                                    break;

                                }
                            }
                            replacement++;
                            pagesAdded.push(currentPage);

                        }
                    } else {
                        for (let frame in frames.pages) {
                            if (frames.pages[frame].value === currentPage) {
                                frames.pages[frame].reference = 1;
                                break;
                            }
                        }
                    }


                    this.results.resultTable.push(lodashCloneDeep(frames));
                }
                this.results.pageFaults = pagesAdded.length;
                this.results.replacement = replacement;
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
            },
            getIndexFirstReferenced(frames, pagesReferenced) {
                let indexList = [];
                for (let frame in frames) {
                    indexList.push(pagesReferenced.lastIndexOf(frames[frame]));
                }
                return frames.lastIndexOf(pagesReferenced[Math.min(...indexList)]);
            },
            getPageReferencedLaterIndex(frames, page) {
                let indexList = [];
                for (let frame in frames) {
                    let index = this.pageList.indexOf(frames[frame], page);
                    if (index === -1) {
                        return frame;
                    } else indexList.push(index);
                }
                return frames.lastIndexOf(this.pageList[Math.max(...indexList)]);
            },
            inFrameClock(currentPage, frames) {
                for (let frame = 0; frame < this.nFrames; frame++) {
                    if (frames.pages[frame].value === currentPage) return true;
                }
                return false;
            },
            increaseClockPointer(pointer) {
                return (pointer + 1) % (this.nFrames)
            }
        }
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .methods > label {
        margin-right: 10px;
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

    .badge {
        margin: 1px;
    }
</style>
