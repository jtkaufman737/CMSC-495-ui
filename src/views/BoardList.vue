<template>
  <div class="boardlist">
    <h5 class="text-center">Explore Fault Tree Analysis boards</h5>
    <table>
      <tr class="row">
        <th class="col">Name</th>
        <th class="col">Description</th>
      </tr>
      <tr v-for="board in boardList" :key="board.id" class="row">
        <td class="col"><a @click="runNav(board.id)">{{ board.name }}</a></td>
        <td class="col">{{ board.description }}</td>
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
      boardList:[]
    }
  },
  methods:{
    runNav(id) {
      this.$router.push(`board/${id}`)
    },
    getBoards() {
      axios.get(`${this.api}/boards`).then(res => {
        this.boardList = res.data 
      }).catch(err => {
        console.error(err)
      })
    }
  },
  mounted() {
    this.getBoards()
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
</style>
