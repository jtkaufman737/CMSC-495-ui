<template>
  <div class="boardlist">
    <h5 class="text-center">Explore Fault Tree Analysis boards</h5>
      <table style="margin:auto">
        <tr class="row">
          <th class="col">Name</th>
          <th class="col">Description</th>
          <th class="col">Edit</th>
          <th class="col">Delete</th>
        </tr>
        <tr v-for="board in boardList" :key="board.id" class="row">
          <!-- col 1 v-if --> 
          <td class="col center" v-if="editBoard.id == board.id">
            <input type="text" v-model="editBoard.name"/>
          </td> 
          <td class="col center" v-else>
            <a @click="runNav(board.id)">{{ board.name }}</a>
          </td>
          <!-- col 2 v-if --> 
          <td class="col center" v-if="editBoard.id == board.id">
            <input type="text" v-model="editBoard.description"/>
          </td> 
          <td class="col center" v-else>
            {{ board.description }}
          </td>
          <td class="col center">
            <button v-if="editBoard.id == board.id" @click="saveBoard(board.id)">Save</button>
            <button v-else @click="editBoard = board">Edit</button>
          </td>
          <td class="col center">
            <button @click="deleteBoard(board.id)">Delete</button>
          </td>
        </tr>
        <tr class="row" v-if="newBoard">
          <td class="col center">
            <input type="text" v-model="editBoard.name"/>
          </td> 
          <td class="col center">
            <input type="text" v-model="editBoard.description"/>
          </td> 
          <td class="col center">
            <button @click="createBoard()">Save</button>
          </td>
          <!-- ---- = silly hack because ran out of time for CSS --> 
          <td class="col center">
            <a span style="visibility:hidden">------------------------</a>
          </td>
        </tr>
        <tr v-else>
          <br/><br/>
          <center>
            <button @click="prepNewBoard()">Create New</button>
          </center>
        </tr>
      </table>
  </div>
</template>
<script>
import axios from 'axios';

export default {
  name: 'BoardList',
  data() {
    return {
      api: process.env.VUE_APP_API_URL,
      boardList:[],
      editBoard: { name: '', description: ''},
      newBoard: false,
    }
  },
  methods:{
    runNav(id) {
      this.$router.push(`board/${id}`)
    },
    async getBoards() {
      axios.get(`${this.api}/boards`).then(res => {
        this.boardList = res.data 
      }).catch(err => {
        console.error(err)
      })
    },
    prepNewBoard() {
      this.editBoard.name = ""
      this.editBoard.description = "" 
      this.newBoard = true 
    },
    async saveBoard(id) {
      await axios.patch(`${this.api}/boards/${id}`,
      {
        name: this.editBoard.name, 
        description: this.editBoard.description
      }).then(async() => {
        await this.getBoards()
        this.editBoard = { name: '', description: '' }
      }).catch(err => console.log(err))
    },
    async createBoard() {
      await axios.post(`${this.api}/boards`,
      {
        name: this.editBoard.name,
        description: this.editBoard.description
      }).then(async res => {
        await this.getBoards()
        this.newBoard = false 
      }).catch(err => {
        console.log(err)
      })
    }, 
    async deleteBoard(id) {
      await axios.delete(
        `${this.api}/boards/${id}`
      ).then(async() => await this.getBoards()
      ).catch(err => console.log(err))
    }
  },
  async mounted() {
    await this.getBoards()
  }
}
</script>
<style>
div.boardlist {
  margin:2em;
  padding:0 5% 0 5%;
}

.col {
  min-width:200px;
}

.col td {
  margin-left:10px;
}

a:hover {
  cursor: pointer;
}

[type="submit"], button {
  width:160px;
}
</style>
