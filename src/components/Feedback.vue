
<template>
  <codemirror v-model="text" :options="cmOption" />
</template>

<script>
import { codemirror } from "vue-codemirror";

// import base style
import "codemirror/lib/codemirror.css";
import "codemirror/mode/scheme/scheme";
import "codemirror/addon/edit/matchbrackets.js";
// theme css

// require active-line.js
import "codemirror/addon/selection/active-line.js";

export default {
  name: "Feedback",
  components: {
    codemirror
  },
  props: {
    text: String
  },
  data() {
    return {
      cmOption: {
        tabSize: 4,
        line: true,
        mode: "text/x-scheme",
        theme: "default",
        matchBrackets: true,
        readOnly: true,
      }
    };
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
codemirror {
  text-align: left;
  margin: 0;
  height: 500px;
}
</style>
