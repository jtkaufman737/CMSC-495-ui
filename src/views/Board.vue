<template>
  <div class="board">
    <AddNodeModal 
      v-if=modalVisible
      v-show=modalVisible
      @closeModal=closeModal
      :allNodes=nodes
      :selectedNode=modalNode
      :nodeParent=nodeParent
      :active=modalVisible
      :edges=edges
      :board_id=board_id
    />
    <h5 class="text-center">{{ boardData.name }}</h5>
    <h6 class="text-center">{{ boardData.description }}</h6>
    <p class="text-center">Click on a node to add children, edit, or delete it. (Or if there are no nodes yet, click anywhere below to start!)</p>
    <div id="cy"></div>
  </div>
</template>

<script>
import axios from "axios";
import cy from 'cytoscape';
import AddNodeModal from "@/components/AddNodeModal";

export default {
  name: 'Board',
  components: { "AddNodeModal": AddNodeModal },
  data () {
    return {
      boardData: { name: '', description: '' },
      board_id: this.$route.params.id,
      api: process.env.VUE_APP_API_URL,
      edges: [],
      nodes: [],
      modalVisible: false,
      modalNode: null,
      nodeParent:""
    }
  },
  methods: {
    async getBoard() {
      await axios.get(`${this.api}/boards`).then(res => {
        this.boardData = res.data.filter(board => {
          return board.id == this.board_id
        })[0]
      }).catch(err => {
        console.log(err)
      })
    },
    async getSymbols() {
      let formattedNodes = []

      await axios.get(`${this.api}/symbols?board_id=${this.board_id}`).then(res => {
        let nodes = res.data 

        nodes.map(node => {
          formattedNodes.push({
            data: {
              id: node.id.toString(),
              name: node.name, 
              description: node.description
            }
          })
        })
      }).catch(err => {
        console.log(err)   
      })

      this.nodes = formattedNodes
    },
    async getConnections() {
      let formattedEdges = []

      await axios.get(`${this.api}/symbol-connections`).then(res => {
        let edges = res.data 

        edges.map(edge => {
          formattedEdges.push({
            data: {
              ref: edge.id, // calling the PK "ref" because ID is a field cytoscape needs constructed of start node id and end node id 
              id: `${edge.start_symbol}${edge.destination_symbol}`,
              source: edge.start_symbol.toString(), 
              target: edge.destination_symbol.toString()
            }
          })
        })
      }).catch(err => {
        console.log(err)
      })

      this.edges = formattedEdges 
    },
    async buildTree() {
      try {
        let cytoscape = cy({
          container: document.getElementById("cy"),
          userZoomingEnabled: false,
          elements: this.combineElements(),
          layout: {
            name: 'breadthfirst',
            directed: 'true'
          },
          style: [ // the stylesheet for the graph
            {
              selector: 'node',
              style: {
                'shape': 'round-rectangle',
                'border-width':'20px',
                'border-style':'solid',
                'border-color':'#104cfb',
                'border-opacity':'0.5',
                'background-color':'white',
                'height': '400px',
                'width': '400px',
                'label': 'data(description)',
                'text-wrap': 'wrap',
                'text-halign': 'center',
                'text-valign': 'center',
                'padding':'15px',
                'font-family':'-apple-system,BlinkMacSystemFont,Avenir,"Avenir Next","Segoe UI",Roboto,Oxygen,Ubuntu,Cantarell,"Fira Sans","Droid Sans","Helvetica Neue",sans-serif',
                'font-size':'3.5em',
              }
            },
          ]
        })

        cytoscape.on("click", (e) => {
          this.raiseModal(JSON.parse(JSON.stringify(e.target._private.data)))
        })
      } catch(e) {
        console.log(e)
      }
    },
    combineElements() {
      return this.nodes.concat(this.edges)
    },
    raiseModal(node) {
      let parent = this.getParentNode(node)

      this.modalNode = node 

      if(parent) this.nodeParent = parent 
      
      this.modalVisible = true 
    }, 
    closeModal(refreshGraph) {
      this.modalVisible = false 

      if(refreshGraph) {
        this.reloadAllData()
      }
    },
    getParentNode(node) {
      let edge = this.edges.filter(edge => {
        return edge.data.target == node.id 
      })[0]

      if(edge) {
        let parent = this.nodes.filter(node => {
          return node.data.id == edge.data.source 
        })[0]

        return parent
      } 
    },
    async reloadAllData() {
      await this.getBoard()
      await this.getSymbols()
      await this.getConnections()
      await this.buildTree()
    }
  },
  async created() {
    // Don't want to build the tree until loaded 
    await this.reloadAllData()
  },
}
</script>
<style scoped>
div#cy {
  min-height:600px;
}
</style>