<template>
  <div>
    <b-container fluid class="lining">
      <b-row align-v="center">
        <b-col cols="7">
          <h2 class="title">
            <b-link to="https://github.com/rymaju/baby-racket">baby-racket</b-link>
            <span v-if="kanren" style="color:crimson">+kanren</span>
          </h2>
        </b-col>
        <b-col>
          <b-icon-play-fill style="width:3em; height:3em;" class="icon" @click="run" />

          <b-icon-puzzle
            class="icon"
            v-if="!kanren"
            style="width:2.5em; height:2.5em; margin:0.3em; color:crimson"
            @click="kanren=!kanren"
          />
          <b-icon-puzzle-fill
            class="icon"
            v-else
            style="width: 2.5em; height:2.5em; margin:0.3em; color:crimson"
            @click="kanren=!kanren"
          />
        </b-col>
      </b-row>
      <b-row>
        <b-col md="7">
          <codemirror v-model="code" :options="editorOptions" />
        </b-col>
        <b-col>
          <codemirror v-model="feedback" :options="feedbackOptions" />
          <codemirror :options="interactOptions" @keyHandled="keyHandled" />
        </b-col>
      </b-row>
    </b-container>
    <b-container class="mt-5">
      <h3>docs</h3>
      <small>baby-racket is a weak infantile language, and may break or produce unexpected behavior. Its very carefree, and gets confused between strings, symbols, and variables often. That said, it is surprisingly smart sometimes.</small>
      <br />

      <h4>built-in</h4>
      <small>
        If the function exists in Racket you have a 50% chance of it existing here. map, filter, foldr, foldl, andmap, ormap, and all your favorite functions are
        included too.
      </small>
      <div style="color:crimson">
        <h4>kanren</h4>
        <small>
          Uses mykanren (a implementation of minikanren made by yours truly) and its behavior fully. For a more detailed breakdown go to the repo
          <a
            href="https://github.com/rymaju/mykanren"
          >here</a>.
        </small>
        <small>
          <b>
            <ul>
              <li>run</li>
              <li>run*</li>
              <li>==</li>
              <li>defrel</li>
              <li>disj2</li>
              <li>conj2</li>
              <li>disj</li>
              <li>conj</li>
              <li>fresh</li>
              <li>conde</li>
              <li>conda</li>
              <li>condu</li>
              <li>success</li>
              <li>fail</li>
            </ul>
          </b>
          Also included are these basic list relations (taken directly from mykanren/examples.rkt)
          <b>
            <ul>
              <li>conso</li>

              <li>caro</li>
              <li>cdro</li>
              <li>nullo</li>
              <li>appendo</li>
            </ul>
          </b>
        </small>
      </div>
      <p class="mb-4">
        Made by
        <b-link to="https://ryanjung.dev">Ryan Jung</b-link>, B.S. Computer Science @ Northeastern University 2023 -
        <b-link to="https://www.github.com/rymaju/baby-racket-ide">See the code for this app here!</b-link>
      </p>
    </b-container>
  </div>
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

import { evaluate, prettify, STANDARD_ENV } from "baby-racket";

export default {
  name: "App",
  components: {
    codemirror
  },
  methods: {
    keyHandled(cm, name) {
      if (!this.evaluating && name === "Enter") {
        let val;
        try {
          val = evaluate(cm.getValue(), {
            env: this.environment,
            kanren: this.kanren
          });
        } catch (err) {
          val = err;
          console.error(err);
        }
        if (val !== undefined) {
          this.feedback +=
            (this.feedback.length > 0 ? "\n" : "") +
            "> " +
            cm.getValue().trim() +
            "\n" +
            prettify(val);
        }
        cm.setValue("");
      }
    },
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
      this.evaluating = true;
      let output = "";

      new Promise((resolve, reject) => {
        try {
          const v = evaluate("(list" + cleanedCode + ")", {
            env: this.environment,
            kanren: this.kanren
          });
          resolve(v);
        } catch (err) {
          reject(err);
        }
      })
        .then(evaluations => {
          for (let statement of evaluations) {
            if (statement !== undefined) {
              output += "> " + prettify(statement) + "\n";
            }
          }
        })
        .catch(err => {
          output += err;
          console.error(err);
        })
        .finally(() => {
          console.log("finished evaluating");
          this.feedback = output;
          this.evaluating = false;
        });
    }
  },
  data() {
    return {
      environment: STANDARD_ENV.clone(),
      evaluating: false,

      kanren: false,
      code:
        "(define fib\n  (lambda (n)\n    (if (< n 2)\n        1\n        (+ (fib (- n 1)) (fib (- n 2))))))\n(fib 20)\n\n(check-equal? (fib 4) 5)\n(check-equal? (fib 7) 21)\n;; (check-equal? (fib 10) 890)\n\n;; to enable minikanren press the red button, then uncomment and run the expression below! \n;; (run 2 (q w) (appendo q w '(1 2 3 4 5)))",
      feedback: "> Click the green Run button to evaluate your code!",
      editorOptions: {
        tabSize: 4,
        styleActiveLine: true,
        lineNumbers: true,
        line: true,
        mode: "text/x-scheme",
        theme: "default",
        matchBrackets: true,
        viewportMargin: Infinity,
        lineWrapping: true
      },
      feedbackOptions: {
        tabSize: 4,
        line: true,
        mode: "text/x-scheme",
        theme: "default",
        matchBrackets: true,
        readOnly: "nocursor",
        viewportMargin: Infinity,
        lineWrapping: true
      },
      interactOptions: {
        tabSize: 4,
        line: true,
        mode: "text/x-scheme",
        theme: "default",
        matchBrackets: true,
        viewportMargin: Infinity,
        lineWrapping: true
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
