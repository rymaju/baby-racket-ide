<template>
  <div>
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
          <codemirror :options="interactOptions" @keyHandled="keyHandled" />
        </b-col>
      </b-row>
    </b-container>
    <b-container class="mt-5">
      <b-row>
        <b-col sm="4">
          <h3>about</h3>
          <p>
            <b-link to="https://github.com/rymaju/baby-racket">baby-racket</b-link>
            {{ }}is a subset of Racket created entirely in Javascript without external dependencies.
            The goal of this project is to create a suitable substitute language for
            Racket's Student Languages
            that can run in the browser. This is mainly an exercise in writing interpreters, if you actually want to run Racket in the brower fast then use webassembly.
          </p>
        </b-col>

        <b-col sm="4">
          <h4>built-in</h4>
          <p>
            If the function exists in Racket you have a 50% chance of it existing here. map, filter, foldr, foldl, andmap, ormap, and all your favorite functions are
            included too. Almost any legal code in a student language is legal code in baby-racket. (Except for big bang and image libraries.)
          </p>
        </b-col>
        <b-col sm="4">
          <h4>fair warning</h4>
          <p>
            baby-racket is a weak infantile language, and may break or produce unexpected
            behavior. Its very carefree, and gets confused between strings, symbols, and
            variables often. That said, it can be surprisingly smart sometimes. Play around with it in the browser. Try and see if your favorite functional program works in baby-racket!
          </p>
        </b-col>
      </b-row>

      <p class="mb-3 mt-5">
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

import { evaluateFile, evaluate, STANDARD_ENV } from "baby-racket";
import dedent from "dedent";

export default {
  name: "App",
  components: {
    codemirror
  },
  methods: {
    run() {
      console.log("evaluating...");
      this.evaluating = true;

      let evaluations = [];

      this.environment = STANDARD_ENV.clone();
      try {
        evaluations = evaluateFile(this.code, { env: this.environment });
        evaluations = evaluations.filter(s => s.length !== 0);
        evaluations = evaluations.map(s => "> " + s);
        this.feedback = evaluations.join("\n");
        console.log("finished evaluating!");
      } catch (e) {
        this.feedback = e.message;
        console.log("error occurred while evaluating!");
        console.error(e);
      }
    },
    keyHandled(cm, name) {
      if (name === "Enter") {
        let val;
        try {
          val = evaluate(cm.getValue(), {
            env: this.environment
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
            val;
        }
        cm.setValue("");
      }
    }
  },
  data() {
    return {
      environment: STANDARD_ENV.clone(),
      code: dedent`(define fib
                      (lambda (n)
                          (if (< n 2)
                              1
                              (+ (fib (- n 1)) (fib (- n 2))))))
                    
                    (fib 20)
                    
                    (check-expect (fib 4) 5)
                    (check-expect (fib 7) 21)

                    ;; uncomment the line below to see what happens when a test fails!
                    ;; (check-expect (fib 10) "a wrong answer")

                    #| multi 
                       line comments are supported but sadly #; are not (yet)
                    |#
                    
                    (cons 'cons (cons 'is (cons 'supported empty)))
                    '(so is quotation)
                    \`(and ,(string-append "quasi" "quotes") and unquotes)

                    (filter (lambda (x) (not (string=? x "garbage"))) '("list abstractions " "garbage" "are supported too!"))

                    (define-struct person [first last uni grad-year])
                    \`(created by ,(make-person 'Ryan 'Jung 'Northeastern 2023))
                    `,

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
        readOnly: true,
        cursorHeight: 0,
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
        lineWrapping: true,
        autofocus: true,
        styleActiveLine: true
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
