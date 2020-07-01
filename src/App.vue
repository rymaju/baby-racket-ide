<template>
  <b-container fluid class="lining">
    <b-row align-v="center">
      <b-col cols="7">
        <h2 class="title">
          <b-link to="https://github.com/rymaju/baby-racket">baby-racket</b-link>
        </h2>
      </b-col>
      <b-col>
        <b-icon-play-fill style="width:3em; height:3em;" class="icon" @click="run" />
      </b-col>
    </b-row>
    <b-row>
      <b-col md="7">
        <codemirror v-model="code" :options="editorOptions" />
      </b-col>
      <b-col>
        <codemirror v-model="feedback" :options="feedbackOptions" />
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import { codemirror } from "vue-codemirror";
// import base style
// TODO svg animation, lambda blood contract, flows into pentagram thing
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
      if (this.evaluating) {
        return;
      }

      let cleanedCode = "";

      for (let line of this.code.split("\n")) {
        if (!line.startsWith(";")) {
          cleanedCode += line + " ";
        }
      }
      console.log("evaluating...");

      let output = "";

      new Promise((resolve, reject) => {
        try {
          const v = evaluate("(list" + cleanedCode + ")");
          resolve(v);
        } catch (err) {
          reject(err);
        }
      })
        .then(evaluations => {
          for (let statement of evaluations) {
            if (statement !== undefined)
              output += "> " + prettify(statement) + "\n";
          }
        })
        .catch(err => {
          output += err;
        })
        .finally(() => {
          console.log("finished evaluating");
          this.feedback = output;
        });
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
        viewportMargin: Infinity
      },
      feedbackOptions: {
        tabSize: 4,
        line: true,
        mode: "text/x-scheme",
        theme: "default",
        matchBrackets: true,
        readOnly: true,
        viewportMargin: Infinity
      }
    };
  }
};

/**
 *
 * TODO:
 * swap play button with spinner and stop concurrent evals
 *  write readmes
 * turn this into a component (useful when we embed into my website later)
 *
 */
</script>

<style>
.title {
  font-weight: 700;
  color: black;
}
.CodeMirror {
  border-left: 1px solid #eee;
  height: auto;
}
b-icon-play-fill:hover {
  cursor: pointer;
}
.icon {
  color: limegreen;
}
.icon:hover {
  cursor: pointer;
}
.icon:active {
  color: green;
}
.lining {
  border: 1px solid #eee;
}
</style>
