<template>
  <div class="static grid grid-flow-row grid-rows-3 w-screen h-screen">
    <div
      class="flex flex-col items-stretch flex-shrink-0 bg-white row-span-3"
      id="main"
    >
      <div
        class="flex items-center flex-shrink-0 p-6 border-b-2 border-stone-700"
      >
        <input
          class="text-md font-semibold"
          v-model="username"
          placeholder="Enter username"
        />
      </div>
      <div
        class="static flex flex-col max-w-full max-h-[88%] p-1 overflow-y-auto"
        id="messageBox"
      >
        <div
          class="h-auto w-full py-3 lh-tight border-b-2"
          v-for="message_ in messages"
          :key="message_"
        >
          <div class="ml-4" id="textContent">
            <div class="flex w-96 align-items-center justify-content-between">
              <strong class="mb-2">{{ message_.username }}</strong>
            </div>
            <div class="grid-cols-10 mb-1 text-sm">{{ message_.message }}</div>
          </div>
        </div>
      </div>
    </div>
    <div class="h-fit relative bottom-0 row-start-4" id="inputBox">
      <form @submit.prevent="submit">
        <input
          class="form-control"
          placeholder="Write a message"
          v-model="message"
        />
      </form>
    </div>
  </div>
</template>

<script src="https://js.pusher.com/7.2/pusher.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

<script>
import Vue from "vue";
import OPENAI_API_KEY from '~/text/key.txt'

const { Configuration, OpenAIApi } = require("openai");
const configuration = new Configuration({
  apiKey: OPENAI_API_KEY,
});
const openai = new OpenAIApi(configuration);

let curUser = "";

export default Vue.extend({
  data() {
    return {
      username: "",
      message: "",
      messages: [],
    };
  },
  methods: {

    async submit() {
      let data = { message: this.message, username: this.username };
      this.messages.push(data);
      curUser = this.message;
      this.message = "";
      this.reConv();
    },

    async gpt(prompt, temp = 0.7, freq = 0, pres = 0, tok = 128) {
      const response = await openai.createCompletion({
        model: "text-davinci-002",
        prompt: prompt,
        max_tokens: tok,
        temperature: temp,
        frequency_penalty: freq,
        presence_penalty: pres,
      });

      return response.data.choices[0].text;

    },

    pushResp(completion){
        
      let GPdaTa = { message: completion, username: "Calypso" };
      this.messages.push(GPdaTa);
    },

    async reConv() {
      let conversation = [];

      this.messages.forEach((element) => {
        conversation.push(`${element.username}: ${element.message}`);
      });
      this.optConvo((conversation));
    },
    
    async optConvo(context = []) {
      let contextString = context.join("\n");
      let summary = "";
      if (context.length > 1) {
        let prompt = `${contextString} \n Detailed summary about conversation between the user ${this.username} and Calypso: `;
        summary = await this.gpt(
          (prompt),
          (0),
          (0.2),
          (0.2),
          (256)
        );
      }
      console.log(contextString);
      console.log(context);
      console.log(curUser);
      console.log(summary);
      await this.cntPrompt(context, summary);
    },

    async cntPrompt(curconv = [], summary) {
      let lastStrings = "";
      if (curconv.length > 2) {
        let lastlines = curconv.slice(-2).join('\n');
        lastStrings = lastlines;  
        console.log(lastStrings);
      }else{
        lastStrings = curconv.join('\n');
      }

      let ctxprompt = `The woman named Calypso named is a soft-spoken, compassionate, and intelligent therapist. Calypso responds with lexically aligned and relevant responses.\n\nCONTEXT:\n${summary}\n${lastStrings}\nCALYPSO:`;

      let response = await this.gpt(
        ctxprompt,
        (0.9),
        (1.5),
        (2),
        (256),
      );

      this.pushResp(response);


    },
  },
});
</script>
