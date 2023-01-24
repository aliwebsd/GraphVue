<template>
  <v-card class="graph-card overflow-hidden" color="#fff" elevation="3" min-height="368">
    <v-system-bar
      color="indigo darken-2"
      dark
    >
      <v-spacer></v-spacer>
      <v-icon>mdi-window-minimize</v-icon>
      <v-icon>mdi-window-maximize</v-icon>
      <v-icon>mdi-close</v-icon>
    </v-system-bar>
    <v-toolbar
      color="indigo"
      dark
    >
      <v-icon class="mr-2">mdi-graph</v-icon>
      <v-toolbar-title>Graph View</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-tooltip bottom>
        <template v-slot:activator="{ on, attrs }">
          <v-btn icon @click="drawer = !drawer" v-on="on" v-bind="attrs">
            <v-icon>{{ paths.length ? 'mdi-filter' : 'mdi-filter-outline' }}</v-icon>
          </v-btn>
        </template>
        <span>Filters</span>
      </v-tooltip>
    </v-toolbar>
    <v-navigation-drawer
        v-model="drawer"
        absolute 
        right
        floating
        temporary
        class="graph__aside">
        <template v-slot:prepend>
          <v-list-item>
            <v-list-item-content>
              <div class="d-flex align-center">
                <v-list-item-title>
                  <v-icon class="mr-2">mdi-filter-outline</v-icon>
                  <span>Graph Path Filter</span>
                </v-list-item-title>
                <v-spacer></v-spacer>
                <v-btn @click="drawer = false" icon>
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </div>
            </v-list-item-content>
          </v-list-item>
        </template>
        <v-divider></v-divider>
        <v-container>
          <v-row>
            <v-col cols="12">
              <v-combobox
                v-model="pathFilter.from"
                :items="fromNodes"
                chips
                clearable
                label="From Node"
                hide-details="auto"
                solo
                outlined
              >
                <template
                  v-slot:selection="{ attrs, item, select, selected }"
                >
                  <v-chip
                    v-bind="attrs"
                    :input-value="selected"
                    close
                    @click="select"
                    @click:close="removeChild('from')"
                  >
                    <strong>{{ item }}</strong>
                  </v-chip>
                </template>
              </v-combobox>
            </v-col>
            <v-col cols="12">
              <v-combobox
                v-model="pathFilter.to"
                :items="toNodes"
                chips
                clearable
                label="To Node"
                hide-details="auto"
                solo
                outlined
              >
                <template
                  v-slot:selection="{ attrs, item, select, selected }"
                >
                  <v-chip
                    v-bind="attrs"
                    :input-value="selected"
                    close
                    @click="select"
                    @click:close="removeChild('to')"
                  >
                    <strong>{{ item }}</strong>
                  </v-chip>
                </template>
              </v-combobox>
            </v-col>
            <v-col cols="12" sm="12" md="6">
              <v-btn color="blue darken-1" block text rounded @click="restGraph">
                <v-icon class="mr-1" small>mdi-filter-remove</v-icon>
                <span>Reset</span>
              </v-btn>
            </v-col>
            <v-col cols="12" sm="12" md="6">
              <v-btn color="primary" block rounded @click="filterPath" :disabled="!pathFilter.from || !pathFilter.to">
                <v-icon class="mr-1" small>mdi-filter-plus-outline</v-icon>
                <span>Filter</span>
              </v-btn>
            </v-col>
          </v-row>
        </v-container>
      </v-navigation-drawer>
      <div class="d-flex flex-column flex-grow-1 h-100">
        <div class="graph__container pa-8 flex-grow-1" v-show="!pathNotFound">
          <div
            class="graph__row d-flex align-center justify-center"
            v-for="row in layout"
            :key="row.id"
          >
            <template v-for="node in row.items">
              <v-tooltip bottom :key="node">
                <template v-slot:activator="{ on, attrs }">
                  <div
                    class="graph__node fade"
                    :id="'node-' + node"
                    :class="{'fade--out': nodesInPaths.length && !nodesInPaths.includes(node)}"
                    v-bind="attrs"
                    v-on="!nodesInPaths.length || nodesInPaths.includes(node) ? on : {}"
                  >
                    {{ node | alis }}
                  </div>
                </template>
                <span>{{ node }}</span>
              </v-tooltip>
            </template>
          </div>
          <div class="d-flex align-center">
            <v-btn color="blue darken-1" text rounded @click="restGraph" v-if="paths.length">
              <v-icon class="mr-2">mdi-filter-remove</v-icon>
              <span>Clear Filter</span>
            </v-btn>
          </div>
          <div class="graph__edges">
            <graph-edge class="fade" v-for="(edge, index) in edges" :key="index" v-bind="edge" ref="graphEdge" :class="{'fade--out': nodesInPaths.length && !(nodesInPaths.includes(edge.source) && nodesInPaths.includes(edge.target))}" :color="(edgesColor[edge.source] && edgesColor[edge.source][edge.target]) || '#7e7e7e'" />
          </div>
        </div>
        <transition name="slide-fade">
          <div class="graph-not-found d-flex flex-column flex-grow-1 align-center justify-center pa-8" v-if="pathNotFound">
            <div class="graph__icon-wrapper">
              <v-icon class="graph__404-icon" size="80" color="#eee">mdi-graph-outline</v-icon>
              <span class="graph__error-code">4 4</span>
            </div>
            <span class="graph__404-text">No Way From <strong>{{ pathFilter.from }}</strong> to <strong>{{ pathFilter.to }}</strong>!</span>
            <v-btn color="blue darken-1" text rounded @click="restGraph">
              <v-icon class="mr-2">mdi-filter-remove</v-icon>
              <span>Clear Filter</span>
            </v-btn>
          </div>
        </transition>
      </div>
  </v-card>
</template>

<script>
import Vue from "vue";
import GraphEdge from "./GraphEdge";
export default Vue.extend({
  name: "GraphView",
  data: () => ({
    drawer: false,
    pathFilter: {
      from: null,
      to: null
    },
    paths: [],
    nodesInPaths: [],
    edgesColor: [],
    pathNotFound: false
  }),
  props: {
    graph: {
      type: Array,
      required: true
    }
  },
  computed: {
    /**
     * Return Parents node: parents list
     * @returns { object }
     */
    parents () {
      const parents = {};
      this.graph.forEach((node) => {
        if (!parents[node.name]) {
          parents[node.name] = []
        }
        node.children.forEach((child) => {
          if (parents[child]) {
            parents[child].push(node.name);
          } else {
            parents[child] = [node.name];
          }
        });
      });
      return parents
    },
    /**
     * Return Layout for print Nodes
     * @returns { array }
     */
    layout () {
      const layout = [
        {
          id: 1,
          items: []
        }
      ]
      const nodes = []
      for (let currentRow = 1; nodes.length < this.graph.length; currentRow++) { // loop on 
        if (!nodes.length && !layout[currentRow - 1].items.length) { // find root nodes on row 1
          for (const m in this.parents) {
            if (this.parents[m].length == 0) {
              layout[0].items.push(m);
              nodes.push(m)
            }
          }
        }
        if (layout[currentRow - 1] && layout[currentRow - 1].items) {
          layout[currentRow - 1].items.forEach(i => { // find before row items children for next row nodes
            for (const m in this.parents) { 
              this.parents[m].forEach(j => {
                if (i === j && !nodes.includes(m)) {
                  if (layout[currentRow]) {
                    layout[currentRow].items.push(m);
                  } else {
                    layout.push({
                      id: layout[currentRow - 1].id + 1,
                      items: [m]
                    })
                  }
                  nodes.push(m)
                }
              })
            }
          })
        }
      }
      return layout;
    },
    /**
     * Return Edges Array
     * @returns { array }
     */
    edges () {
      const edges = [];
      for (const i in this.parents) {
        this.parents[i].forEach((j) => {
          edges.push({ source: j, target: i })
        })
      }
      return edges
    },
    /**
     * Return Nodes as string Array
     * @returns { Array<string> }
     */
    nodes () {
      return this.graph.map(i => i.name)
    },
    /**
     * Return fromNodes for filter path source input
     * @returns { Array<string> }
     */
    fromNodes () {
      return this.pathFilter.to ? this.nodes.filter(i => i !== this.pathFilter.to) : this.nodes
    },
    /**
     * Return toNodes for filter path target input
     * @returns { Array<string> }
     */
    toNodes () {
      return this.pathFilter.from ? this.nodes.filter(i => i !== this.pathFilter.from ) : this.nodes
    }
  },
  components: {
    GraphEdge
  },
  watch: {
    graph: { 
      handler () {
        this.restGraphEdges()
        localStorage.setItem('graph', JSON.stringify(this.graph))
      },
      deep: true
    }
  },
  methods: {
    removeChild (list) {
      this.pathFilter[list] = null
    },
    restGraphEdges () {
      setTimeout(() => {
        this.$refs.graphEdge.forEach(i => {
          i.init()
        })
      })
    },
    /**
     * Return Paths from source to target
     * @returns { Array<Array<string>> }
     */
    filterPath () {
      let currentNode = this.graph.find(i => i.name === this.pathFilter.from),
      paths = [[]],
      pathIndex = 0,
      nodeIndex = 0,
      nodes = []
      this.pathNotFound = false
      this.paths = []
      this.nodesInPaths = []
      this.edgesColor = {}
      if (currentNode) {
        ;(
        /**
         * Find Paths
         * @returns { Array<string> }
         */
        function findPath (currentNode) { // recursive and IIFE function 
          currentNode.children.forEach(node => {
            if (!nodes.includes(node)) {
              nodes.push(node)
            }
            const nodeData = this.graph.find(i => i.name === node)
            if (node === this.pathFilter.to ) {
              paths[pathIndex][nodeIndex] = node
              pathIndex++
              nodeIndex = 0
            } else if (nodeData.children.length) {
              if (!paths[pathIndex]) {
                paths[pathIndex] = []
              }
              paths[pathIndex][nodeIndex] = node
              nodeIndex++
              findPath.call(this, nodeData)
            } else if (nodeIndex > 0) {
              nodeIndex--
            }
          })
        }).bind(this)(currentNode)
        if (paths[paths.length - 1][paths[paths.length - 1].length - 1] !== this.pathFilter.to) {
          paths.pop()
        }
        this.drawer = false
        if (!paths.length) {
          this.pathNotFound = true
          return
        }
        paths.forEach(i => {
          i.unshift(this.pathFilter.from)
          const color = '#'+(Math.random()*0xFFFFFF<<0).toString(16)
          i.forEach((j, jIndex) => {
            if (this.nodesInPaths.indexOf(j) === -1) {
              this.nodesInPaths.push(j)
            }
            if (i[jIndex+1]) {
              if (!this.edgesColor[j]) {
                this.edgesColor[j] = {}
              }
              this.edgesColor[j][i[jIndex+1]] = color
            }
          })
        })
        this.paths = paths
      }
    },
    restGraph () {
      this.paths = []
      this.nodesInPaths = []
      this.edgesColor = {}
      this.pathNotFound = false
      this.pathFilter = {
        from: null,
        to: null
      }
      this.drawer = false
      setTimeout(() => {
        this.restGraphEdges()
      })
    }
  },
  filters: {
    alis: function (value) {
      return value.split('-').map(i => i.substr(0, 1).toUpperCase()).join('')
    }
  }
})
</script>

<style lang="scss">
.graph {
  &__container {
    z-index: 0;
    position: relative;
  }
  &__error-code {
    font-size: 88px;
    position: absolute;
    color: #eee;
    z-index: 0;
    transform: rotate(-45deg);
    font-family: monospace;
  }
  &__node {
    width: 3em;
    height: 3em;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    background-color: #fff;
    border: 1px solid #f9b366;
    color: #747474;
    z-index: 1;
    margin: 1.5em;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  &__edges {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 22px;
    left: 22px;
    z-index: -1;
  }
  &__icon-wrapper {
    width: 160px;
    height: 160px;
    background-color: #00000026;
    border-radius: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    transform: rotate(45deg);
  }
  &__404-icon {
    transform: rotate(-45deg);
  }
  &__404-text {
    color: #888;
    font-size: 1.2rem;
    font-weight: bold;
    font-family: 'Roboto-Regular';
    margin-top: 32px;
  }
}
.fade {
  transition: opacity .3s ease-out;
  &--out {
    opacity: 0;
  }
}
.graph-not-found {
  gap: 8px;
  padding-top: 32px;
}
.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to {
  transform: translateX(10px);
  opacity: 0;
}
</style>
