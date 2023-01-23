<template>
  <div
    onload="onLoad()"
    class="static grid grid-flow-row grid-rows-3 w-screen h-screen"
  >
    <div
      class="flex flex-col items-stretch flex-shrink-0 bg-white row-span-3"
      id="main"
    >
      <div
        class="grid grid-cols-2 items-center flex-shrink-0 p-6 border-b-2 border-stone-700"
      >
        <input
          class="text-md font-semibold"
          v-model="username"
          placeholder="Enter username"
        />

        <div class="place-self-end">
          <a href="/">
            <button
              class="text-sm font-semibold py-2 px-4 rounded-md bg-violet-500 hover:bg-violet-600 text-white"
              @click="endSession()"
            >
              End Session
            </button></a
          >
        </div>
      </div>

      <div
        class="static flex flex-col max-w-full max-h-[88%] p-1 overflow-y-auto"
        id="messageBox"
      >
        <div
          class="h-auto w-full py-3 lh-tight border-b-2"
          v-for="(message_, i) in messages"
          :key="message_"
        >
          <div class="ml-4" id="textContent">
            <div class="flex w-96 align-items-center justify-content-between">
              <strong class="mb-2">{{ message_.username }}</strong>
            </div>
            <div class="grid-cols-10 mb-1 text-sm">{{ message_.message }}</div>
            <div class="" v-if="message_.m_type === 'T'">
              <div>
                <div class="flex flex-row">
                  <label for="yes" class="block p-2 text-gray-700">
                    <input
                      type="radio"
                      id="yes"
                      :name="'vote_' + { i }.i"
                      class="form-radio w-4 h-4 text-indigo-600 transition duration-150 ease-in-out"
                      @click="model_FB(1, { i })"
                    />
                    <svg
                      width="16"
                      height="16"
                      viewBox="0 0 24 24"
                      xmlns="http://www.w3.org/2000/svg"
                    >
                      <path d="M0 0h24v24H0z" fill="none" />
                      <path
                        d="M1 21h4V9H1v12zm22-11c0-1.1-.9-2-2-2h-6.31l.95-4.57.03-.32c0-.41-.17-.79-.44-1.06L14.17 1 7.59 7.59C7.22 7.95 7 8.45 7 9v10c0 1.1.9 2 2 2h9c.83 0 1.54-.5 1.84-1.22l3.02-7.05c.09-.23.14-.47.14-.73v-1.91l-.01-.01L23 10z"
                      />
                    </svg>
                  </label>
                  <label for="no" class="block p-2 text-gray-700">
                    <input
                      type="radio"
                      id="no"
                      :name="'vote_' + { i }.i"
                      class="form-radio w-4 h-4 text-indigo-600 transition duration-150 ease-in-out"
                      @click="model_FB(0, { i })"
                    />
                    <svg
                      width="16"
                      height="16"
                      viewBox="0 0 24 24"
                      xmlns="http://www.w3.org/2000/svg"
                    >
                      <path
                        d="M15 3H6c-.83 0-1.54.5-1.84 1.22l-3.02 7.05c-.09.23-.14.47-.14.73v2c0 1.1.9 2 2 2h6.31l-.95 4.57-.03.32c0 .41.17.79.44 1.06L9.83 23l6.59-6.59c.36-.36.58-.86.58-1.41V5c0-1.1-.9-2-2-2zm4 0v12h4V3h-4z"
                      />
                    </svg>
                  </label>
                </div>
              </div>
            </div>
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
import OPENAI_API_KEY from "~/text/key.txt";

const { Configuration, OpenAIApi } = require("openai");
const configuration = new Configuration({
  apiKey: OPENAI_API_KEY,
});
const openai = new OpenAIApi(configuration);


export default Vue.extend({
  data() {
    return {
      username: "",
      message: "",
      messages: [],
      prompt_resp: [],
      m_type: "",
      feedback: [],
    };
  },
  methods: {
    async submit() {
      this.m_type = "C";
      let data = {
        message: this.message,
        username: this.username,
        m_type: this.m_type,
      };
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
      });

      return response.data.choices[0].text.replace(/(\r\n|\n|\r)/gm, "");
    },

    async cntPrompt(curconv = [], summary) {
      let lastStrings = "";
      if (curconv.length > 2) {
        let lastlines = curconv.slice(-2).join("\n");
        lastStrings = lastlines;
      } else {
        lastStrings = curconv.join("\n");
      }

      let ctxprompt = `The woman named Calypso named is a soft-spoken, compassionate, and intelligent therapist. Calypso responds with lexically aligned and relevant responses relating only to therapy.\n\nCONTEXT:${summary}\n\n${lastStrings}\nCalypso:`;
      console.log(ctxprompt);
      let response = await this.gpt(ctxprompt, 1, 2, 2, 256);

      this.pushResp(response);
    },

    async reConv() {
      let conversation = [];

      this.messages.forEach((element) => {
        conversation.push(`${element.username}: ${element.message}`);
      });
      this.optConvo(conversation);
    },

    async optConvo(context = []) {
      let contextString = context.join("\n");
      let summary = "";
      if (context.length > 2) {
        let prompt = `${contextString} \n Detailed summary about conversation between the user ${this.username} and Calypso: `;
        summary = await this.gpt(prompt, 0, 0.2, 0.2, 256);
      }

      await this.cntPrompt(context, summary.trim());
    },

    pushResp(completion) {
      this.m_type = "T";
      let GPdaTa = {
        message: completion,
        username: "Calypso",
        m_type: this.m_type,
      };
      this.messages.push(GPdaTa);
      //FB_aggregate();
      //console.log(this.messages);
    },

    //FB_aggregate() {},

    model_FB(u_eval, index) {
      let tru_ind = (index.i - 1) / 2;
      if (u_eval == 1) {
        if (tru_ind >= this.feedback.length) {
          this.feedback.push(u_eval);
        } else {
          this.feedback[tru_ind] = u_eval;
        }
        console.log(this.feedback);
        console.log(index);
      } else if (u_eval == 0) {
        if (tru_ind >= this.feedback.length) {
          this.feedback.push(u_eval);
        } else {
          this.feedback[tru_ind] = u_eval;
        }
        console.log(this.feedback);
        console.log(index);
      }
    },

    onLoad(){
      let sessionID = this.$route.query.sessionID;
      this.$data.page.sessionID = sessionID;
      console.log("the page has loaded")
    },

    async endSession(){

      //sends messsages and feedback arrays to backend

    },
  },
});
</script>
