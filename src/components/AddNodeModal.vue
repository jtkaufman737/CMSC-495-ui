<template>
  <div class="modal-backdrop">
    <div class="modal">
        <header class="modal-header">
            <h5 v-if="changeType=='edit'" class="underline">Edit Node</h5>
            <h5 v-else-if="changeType=='createChild'" class="underline">Add New Node</h5>
            <h5 v-else class="underline">Delete Node</h5>
            <button 
                type="button"
                class="btn-close"
                @click="closeModal(false)"
            >x
            </button>
        </header>
        <section class="modal-body">
            <p class="noPad"><strong>Select change type</strong></p>
            <div>
                <span v-for="change in availableChangeTypes" :key="change.name">
                 <input
                    type="radio"
                    :id="change.name"
                    :value="change.name"
                    v-model="changeType"
                  />
                  <label :for="change">{{ change.label }}</label><br/>
                </span>
            </div><br/>
            <div v-if="changeType == 'deleteChild'">
              <p>Warning: deleting this node will delete any children nodes below it</p>
            </div>
            <div v-else>
                <p v-if="changeType == 'edit'" class="marginBottom"><strong>Edit Node</strong></p>
                <p v-else class="marginBottom"><strong>Create New Node</strong></p>
                <p>Parent Node:</p>   
                  <span v-if="allNodes.length > 0">
                    <select 
                      v-model="newNodeParent"
                    >
                      <option 
                        v-for="node in allNodes" 
                        :key=node.data.id
                        :value=node.data
                      >{{ node.data.name}} - {{node.data.description }}</option>
                    </select>
                  </span>
                  <span v-else>
                    <p><em>No parent node available, create the first</em></p>
                  </span>
                <p>Node Type:</p>  
                <select 
                   v-model="currEditNode.symbol_type"
                >
                  <option 
                    v-for="item in symbolTypes"
                    :key=item.id 
                    :value=item.id
                  >{{ item.type }}</option>
                </select>
                <p>Name:</p>
                <input type="text" v-model=currEditNode.name>
                <p>Description:</p>
                <input type="text" v-model=currEditNode.description>
            </div>
            <div>
              <br/><br/>
              <center>
                <button @click="writeNode">Save</button>
              </center>
            </div>
        </section>
    </div>
  </div>
</template>
<script>
import axios from "axios"

export default {
    name: "AddNodeModal",
    props:[
      "selectedNode", 
      "allNodes",
      "nodeParent",
      "active",
      "edges",
      "board_id"
    ],
    data() {
        return {
            symbolTypes: [],
            api: process.env.VUE_APP_API_URL,
            changeType: "createChild",
            currEditNode:{
                name: "",
                description: "",
                symbol_type: "",
                id: ""
            },
            newNodeParent: null,
            changeTypes: [
              { name: "edit", label: "Edit Node"},
              { name: "createChild", label: "Create Node"}, 
              { name: "deleteChild", label: "Delete Node"}
            ],
            changeTypeToAPICall: {
              edit: this.patchExistingNode, 
              createChild: this.postNewNode, 
              deleteChild: this.deleteExistingNode  
            }
        }
    },
    methods: {
        closeModal(refreshGraph) {
            this.$emit("closeModal", refreshGraph)
        },
        async getSymbolTypes() {
            await axios.get(`${this.api}/symbol-types`).then(res => {
                this.symbolTypes = res.data 
            }).catch(err => {
                console.log(err)
            })
        },
        writeNode() {
          this.changeTypeToAPICall[this.changeType]()  
        },
        async postNewNode() {
          console.log("POST RUNNING")
          // let newNodeID

          // // create symbol object
          // await axios.post(`${this.api}/symbols`,
          //   {
          //     name: this.currEditNode.name, 
          //     description: this.currEditNode.description,
          //     symbol_type: this.currEditNode.symbol_type
          //   }
          // ).then(res => newNodeID = res.data.id 
          // ).catch(err => console.log(err))

          // // create appropriate connections, IF not first node
          // if(this.newNodeParent) {
          //   console.log("NODE PARENT RECOGNIZED")
          //   await axios.post(`${this.api}/symbol-connections`,
          //     {
          //       start_symbol: parseInt(this.newNodeParent.id),
          //       destination_symbol: parseInt(newNodeID),
          //       board_id: this.board_id
          //     }
          //   ).then(res => console.log(res.data)).catch(err => console.log(err))
          // }

          // // create symbol/board membership 
          // await axios.post(`${this.api}/board-symbols`,
          //   {
          //     board_id: this.board_id,
          //     symbol_id: newNodeID
          //   }
          // ).catch(err => console.log(err))

          // this.closeModal(true)
        }, 
        async patchExistingNode() {
          console.log("PATCH EXISTING NODE RUNNING")
          // first deal with node itself 
          await axios.patch(`${this.api}/symbols/${this.selectedNode.id}`,
            {
              name: this.currEditNode.name, 
              description: this.currEditNode.description,
              symbol_type: this.currEditNode.symbol_type
            }
          ).then(res => {
            this.closeModal(true)
          }).catch(err => {
            console.log(err)
          })

          // then deal with connections 
          let idToFind = `${this.nodeParent.data.id}${this.currEditNode.id}`   
          let edge = this.edges.find(edge => edge.data.id == idToFind)

          if(edge) {
            await axios.patch(`${this.api}/symbol-connections/${edge.data.ref}`,
              {
                id: edge.ref, 
                start_symbol: parseInt(this.newNodeParent.id),
                destination_symbol: parseInt(this.currEditNode.id) 
              }).then(res => console.log(res.data))
              .catch(err => console.log(err))
          }
        }, 
        async deleteExistingNode() {
          await axios.delete(`${this.api}/symbols/${this.selectedNode.id}`).then(res => {
            this.closeModal(true)
          }).catch(err => {
            console.log(err)
          })
        }
    },
    computed: {
      availableChangeTypes() {
        if(this.allNodes.length) {
          return this.changeTypes 
        } else {
          return [{ name: "createChild", label: "Create Node"}]
        }
      }
    },
    watch: {
      active() {
        if(this.active) {
          this.newNodeParent = this.nodeParent
          this.currEditNode = this.selectedNode
        }
      },
      changeType() {
        if(this.changeType == "createChild") {
          this.currEditNode = {
            name: "",
            description: "",
            symbol_type:""
          }
        } else {
          this.currEditNode = this.selectedNode
        }
      }
    },
    async mounted() {
        await this.getSymbolTypes()
    }
}
</script>
<style>
  .modal-backdrop {
    z-index:4;
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.3);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .marginBottom {
      margin-bottom:5px;
  }

  .noPad {
    padding:0px !important;
    margin:0px !important;
  }

  .modal {
    background: #FFFFFF;
    box-shadow: 2px 2px 20px 1px;
    overflow-x: auto;
    display: flex;
    flex-direction: column;
    min-width:300px;
    min-height:200px;
  }

  h5.underline {
    height:25px;
    margin:5px;
  }

  .modal-header,
  .modal-footer {
    padding:0px 5px 5px 10px;
    display: flex;
    border-bottom:2px lightgray solid;
  }

  .modal-header {
    position: relative;
    border-bottom: 1px solid #eeeeee;
    justify-content: space-between;
  }

  .modal-footer {
    border-top: 1px solid #eeeeee;
    flex-direction: column;
    justify-content: flex-end;
  }

  .modal-body {
    position: relative;
    padding: 25px;
  }

  .btn-close {
    position: absolute;
    top: 0;
    right: 0;
    border: none;
    font-size: 20px;
    padding: 10px;
    cursor: pointer;
    font-weight: bold;
    background: transparent;
  }

  .btn-green {
    border-radius: 2px;
  }

  input[type="text"] {
    width:93%;
  }
</style>
