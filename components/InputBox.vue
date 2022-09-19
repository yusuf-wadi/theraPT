<template>
  <div class="bottom-10 w-5/6 h-1/6" id="inputBox">
    <input
      v-model="message"
      @keyup.enter="sendMessage"
      placeholder="Type a message..."
    />
  </div>
</template>

<script lang="ts">
import Vue from "vue";
export default Vue.extend({
  data() {
    return {
      message: "",
    };
  },
  methods: {
    sendMessage() {
      const message = this.message;
    },
  },
});
</script>
