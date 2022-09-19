<template>
    <div
      class="grid grid-rows-3 grid-cols-2 w-5/6 h-5/6 border-8 rounded-lg"
      id="message-box"
    >
      <Message :Message="Message" />
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Message from "~/components/Message.vue";
import axios from "axios";

export default Vue.extend({
  name: "MessageBox",
  components: { Message },
  props: ["Message"],
  data() {
    return {
      message: "",
    };
  },
  methods: {
    sendMessage() {
      const message = this.message;

      // Create a new Message component with the content of the message
      let messageComponent = new Message();
      messageComponent.$mount(message);

      // Add the new Message component to the list of Message components
      this.$children.push(messageComponent);

      // Clear the message input
      this.message = "";

      // // Get a response from the API and place it in a new Message component
      // axios.get("/api/message", { params: { message } }).then((response) => {
      //   let messageComponent = new Message();
      //   messageComponent.$mount(message);
      //   this.$children.push(messageComponent);
      // });
    },
  },
});
</script>
