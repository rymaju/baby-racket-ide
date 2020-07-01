
<template>
  <codemirror v-model="value" :options="cmOption" />
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
  name: "Editor",
  components: {
    codemirror
  },
  props: ["value"],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >`,
  computed: {
    code: {
      get: function() {
        return this.value;
      },
      set: function(value) {
        this.$emit("listchange", value);
      }
    }
  },
  data() {
    return {
      cmOption: {
        tabSize: 4,
        styleActiveLine: true,
        lineNumbers: true,
        line: true,
        mode: "text/x-scheme",
        theme: "default",
        matchBrackets: true,
        viewportMargin: 20
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
  height: auto;
}
</style>
