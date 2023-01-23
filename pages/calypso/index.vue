<template>
  <body class="flex flex-col justify-center items-center h-screen bg-gray-200">
    <h1 class="text-4xl font-bold text-center text-green-800">Calypso</h1>
    <a @click="new_sess()">
      <button
        class="outline outline-1 hover:outline-2 outline-stone-300 bg-gray-100 hover:bg-gray-400 text-gray-700 font-bold py-2 px-4 rounded-full mt-8"
      >
        Begin Session
      </button>
    </a>
  </body>
</template>

<script lang="ts">
import Vue from "vue";
import { v4 as uuidv4 } from 'uuid';

export default Vue.extend({
  data(){
    return{

    };
  },
  methods:{
    new_sess(){
      this.$router.push({ path: '/calypso/test', query: { sessionID: uuidv4() } })
    }
  }

});
</script>
