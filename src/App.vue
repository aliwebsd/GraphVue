<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="amn-logo"
          class="shrink mr-2"
          contain
          src="@/assets/logo.png"
          transition="scale-transition"
          width="64"
        />
      </div>
      <v-spacer></v-spacer>
    </v-app-bar>
    <v-main>
      <v-container>
        <v-row no-gutters>
          <v-col cols="12">
            <graph-table v-model="graph"></graph-table>
          </v-col>
        </v-row>
        <v-row no-gutters class="mt-10">
          <v-col cols="12">
            <graph-view :graph="graph"></graph-view>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import Vue from 'vue';
import GraphView from './components/GraphView'
import GraphTable from './components/GraphTable';

export default Vue.extend({
  name: 'App',
  components: {
    GraphView,
    GraphTable
  },
  data: () => ({
    graph: [
      {
        id: 1,
        name: 'a',
        description: 'this is a node.',
        children: ['b', 'c', 'd']
      },
      {
        id: 2,
        name: 'b',
        description: 'this is b node',
        children: ['e']
      },
      {
        id: 3,
        name: 'c',
        description: 'this is c node',
        children: ['f']
      },
      {
        id: 4,
        name: 'd',
        description: 'this is d node',
        children: ['f']
      },
      {
        id: 5,
        name: 'e',
        description: 'this is e node',
        children: []
      },
      {
        id: 6,
        name: 'f',
        description: '',
        children: []
      }
    ]
  }),
  methods: {
    onChangeGraph (graph) {
      this.graph = graph
    }
  },
  created () {
    if(localStorage.graph) {
      this.graph = JSON.parse(localStorage.getItem('graph'))
    }
  }
})
</script>
