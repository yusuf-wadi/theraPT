<template>
  <div
    class="row-start-3 col-start-2 border-4 rounded-2xl"
    id="messageContents"
  >
    <div class="text-lg relative top-10 left-10" id="message">
      {{ Message }}
    </div>
  </div>
</template>

<script>
import Vue from "vue";

export default Vue.extend({
  props: ["Message"],
});
</script>
