


<template>
    <div>
      <body class="static">
        <div class="flex flex-col items-center justify-center h-screen">
          <MessageBox/>
          <InputBox/>
        </div>  
      </body>
    </div>
  </template>
  
  <script lang="ts">
  import Vue from "vue";
  import Message from "~/components/Message.vue";
  import axios from "axios";
  import MessageBox from "~/components/MessageBox.vue";
  import InputBox from "~/components/InputBox.vue";
  
  export default Vue.extend({
    name: "IndexPage",
    components: { Message, MessageBox, InputBox },
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
  