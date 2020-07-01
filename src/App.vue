<template>
  <div>
    <b-row>
      <b-col>
        <h2 class="title">baby-racket v1.0.0</h2>
      </b-col>
      <b-col>
        <b-icon-play-fill style="width:2.5em; height:2.5em;" class="text-success" @click="run" />
      </b-col>
    </b-row>
    <b-row no-gutters>
      <b-col style="width:60vw;">
        <codemirror v-model="code" :options="editorOptions" />
      </b-col>
      <b-col style="width:40vw">
        <codemirror v-model="feedback" :options="feedbackOptions" />
      </b-col>
    </b-row>
  </div>
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
import "pattern.css";

import { evaluate, prettify } from "baby-racket";

export default {
  name: "App",
  components: {
    codemirror
  },
  methods: {
    run() {
      let cleanedCode = "";

      for (let line of this.code.split("\n")) {
        if (!line.startsWith(";")) {
          cleanedCode += line + " ";
        }
      }
      console.log("evaluating...");
      const evaluations = evaluate("(list" + cleanedCode + ")");
      console.log("finished evaluating");
      let output = "";
      for (let statement of evaluations) {
        if (statement !== undefined)
          output += "> " + prettify(statement) + "\n";
      }
      this.feedback = output;
    }
  },
  data() {
    return {
      code:
        "(define fib\n    (lambda (n)\n        (if (< n 2)\n            1\n            (+ (fib (- n 1)) (fib (- n 2))))))\n(fib 20)",
      feedback: "> Click the green Run button to evaluate your code!",
      editorOptions: {
        tabSize: 4,
        styleActiveLine: true,
        lineNumbers: true,
        line: true,
        mode: "text/x-scheme",
        theme: "default",
        matchBrackets: true,
        viewportMargin: 20
      },
      feedbackOptions: {
        tabSize: 4,
        line: true,
        mode: "text/x-scheme",
        theme: "default",
        matchBrackets: true,
        readOnly: true
      }
    };
  }
};
</script>

<style>
.title {
  font-weight: 700;
  color: black;
}
</style>
