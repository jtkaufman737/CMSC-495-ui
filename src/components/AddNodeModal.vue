<template>
  <div class="modal-backdrop">
    <div class="modal">
        <header class="modal-header">
            <h5 v-if="changeType=='edit'" class="underline">Edit Node</h5>
            <h5 v-else class="underline">Add New Node</h5>
            <button 
                type="button"
                class="btn-close"
                @click="closeModal"
            >x
            </button>
        </header>
        <section class="modal-body">
            <p class="noPad"><strong>Select change type</strong></p>
            <div>
                <input 
                    type=radio 
                    id="edit" 
                    value="edit" 
                    v-model="changeType"
                />
                <label for="edit">Edit current node</label>
                <br/>
                <input 
                    type=radio 
                    id="createChild"
                    value="createChild"
                    v-model="changeType"
                />
                <label for="createChild">Create child node</label>
            </div><br/>
            <div v-if="changeType == 'edit'">
                <p class="marginBottom"><strong>Edit</strong></p>
                <p>Parent Node:</p>   
                <select v-model="newNodeParent">
                    <option v-for="node in allNodes" :key=node.data.id>{{ node.data.name }}</option>
                </select>
            </div>
            <div v-else>
                <p class="marginBottom"><strong>Create New Node</strong></p> 
            </div>
        </section>
    </div>
  </div>
</template>
<script>
import axios from "axios"

export default {
    name: "AddNodeModal",
    props:["selectedNode", "allNodes","nodeParent"],
    data() {
        return {
            symbolTypes: [],
            api: process.env.VUE_APP_API_URL,
            changeType: "edit",
            currEditNode:{
                name: "",
                description: ""
            },
            newNodeParent: nodeParent
        }
    },
    methods: {
        closeModal() {
            this.$emit.closeModal()
        },
        async getSymbolTypes() {
            await axios.get(`${this.api}/symbol-types`).then(res => {
                this.symbolTypes = res.data 
            }).catch(err => {
                console.log(err)
            })
        }
    },
    async mounted() {
        await this.getSymbolTypes
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
</style>
