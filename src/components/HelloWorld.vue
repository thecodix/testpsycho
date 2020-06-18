<template>
  <div class="hello">
    <!--container-->
    <section class="container">

      <!--questionBox-->
      <div class="questionBox" id="app">

        <!-- transition -->
        <transition :duration="{ enter: 500, leave: 300 }"
                    enter-active-class="animated zoomIn"
                    leave-active-class="animated zoomOut"
                    mode="out-in">

          <!--chooseQuiz-->
          <div v-if="!chosenQuiz"
               v-bind:key="questionIndex"
               class="questionContainer">

            <header>
              <h2 class="title is-6">Elige asignatura</h2>
            </header>

            <!--options-->
            <div class="optionContainer">
              <a class="button actionButton option"
                 v-on:click="chooseSubject(social)">
                Psicología Social
              </a>
              <a class="button actionButton option"
                 v-on:click="chooseSubject(emocion)">
                Psicología de la Emoción
              </a>
            </div>
            <!--/options-->

          </div>
          <!--/chooseQuiz-->

          <!--qusetionContainer-->
          <div class="questionContainer"
               v-if="chosenQuiz && questionIndex<quiz.questions.length"
               v-bind:key="questionIndex">

            <header>
              <h1 class="title is-6">{{title}}</h1>
              <h4>Tests examen 2020</h4>
              <!--progress-->
              <div class="progressContainer">
                <div class="progress">
                  <div class="progress-bar bg-success" role="progressbar"
                       v-bind:style="{ width: (corrects/quiz.questions.length)*100 + '%' }"
                       aria-valuemin="0" aria-valuemax="100">{{corrects}}</div>
                  <div class="progress-bar bg-danger" role="progressbar"
                       v-bind:style="{ width: (incorrects/quiz.questions.length)*100 + '%' }"
                       aria-valuemin="0" aria-valuemax="100">{{incorrects}}</div>
                </div>
                <p>{{questionIndex+1}} de {{quiz.questions.length}} preguntas</p>
                <p>Tiempo restante: {{timeLeft}}</p>
              </div>
              <!--/progress-->
            </header>

            <!-- questionTitle -->
            <p class="titleContainer title">{{ quiz.questions[questionIndex].text }}</p>
            <div class="titleDate"
               v-if="quiz.questions[questionIndex].fecha">
               {{ quiz.questions[questionIndex].fecha.mes }} -
               {{ quiz.questions[questionIndex].fecha.year }}
            </div>

            <!-- quizOptions -->
            <div class="optionContainer">
              <div class="option"
                   v-for="(response, index)
                          in quiz.questions[questionIndex].responses"
                   @click="selectOption(index, response)"
                   :class="{ 'is-correct':
                              userResponses[questionIndex] == index
                              && response.correct,
                              'is-wrong':
                              userResponses[questionIndex] == index
                              && !response.correct}"
                   :key="index">
                {{ index | charIndex }}. {{ response.text }}
              </div>
            </div>

            <!--quizFooter: navigation and progress-->
            <footer class="questionFooter">

              <!--pagination-->
              <nav class="pagination" role="navigation" aria-label="pagination">

                <!-- back button -->
                <a class="button actionButton"
                   v-on:click="prev()"
                   :disabled="questionIndex < 1">
                  Anterior
                </a>

                <!-- next button -->
                <a class="button actionButton" v-on:click="next()"
                   :disabled="questionIndex===quiz.questions.length">
                  {{ (userResponses[questionIndex]==null)?'Saltar pregunta':'Siguiente' }}
                </a>

              </nav>
              <!--/pagination-->

            </footer>
            <!--/quizFooter-->

          </div>
          <!--/questionContainer-->

          <!--quizCompletedResult-->
          <div v-if="chosenQuiz && questionIndex === quiz.questions.length"
               v-bind:key="questionIndex"
               class="quizCompleted has-text-centered">

            <!-- quizCompletedIcon: Achievement Icon -->
            <span class="icon">
                <i class="fa"
                   :class="score()>3
                   ?'fa-check-circle-o is-active':'fa-times-circle'">
                </i>
              </span>

            <!--resultTitleBlock-->
            <h2 class="title">
              ¡Buen trabajo!
            </h2>
            <div class="progressContainer">

              <div class="progress">
                <div class="progress-bar bg-success" role="progressbar"
                     v-bind:style="{ width: (corrects/quiz.questions.length)*100 + '%' }"
                     aria-valuemin="0" aria-valuemax="100">{{corrects}}</div>
                <div class="progress-bar bg-danger" role="progressbar"
                     v-bind:style="{ width: (incorrects/quiz.questions.length)*100 + '%' }"
                     aria-valuemin="0" aria-valuemax="100">{{incorrects}}</div>
              </div>
            </div>
            <p class="subtitle">
              Puntuación total: {{ score() }} / 10
            </p>
            <br>
            <a class="button" @click="restart()">Volver a empezar <i class="fa fa-refresh"></i></a>
            <!--/resultTitleBlock-->

          </div>
          <!--/quizCompetedResult-->

        </transition>

      </div>
      <!--/questionBox-->

    </section>
    <!--/container-->
  </div>
</template>

<script>
import EXAM_SOCIAL_JSON from './json/social2pp.json';
import EXAM_EMOCION_JSON from './json/emocion.json';

const numQuestions = 2;

export default {
  // el: '#app',
  data() {
    return {
      chosenQuiz: false,
      title: '',
      corrects: 0,
      incorrects: 0,
      quiz: {
        user: 'Dave',
        questions: {},
      },
      social: {
        title: 'Psicología Social',
        jsonFile: EXAM_SOCIAL_JSON,
        numQuestions: 20,
        time: 40 * 60, // in seconds
        plusScore: 0.45,
        minusScore: 0.20,
      },
      emocion: {
        title: 'P. de la Emoción',
        jsonFile: EXAM_EMOCION_JSON,
        numQuestions: 40,
        time: 20 * 60, // in seconds
        plusScore: 0.25,
        minusScore: 0.25,
      },
      questionIndex: 0,
      userResponses: Array(numQuestions).fill(null),
      isActive: false,
      form: {
        selected: '',
      },
      myToggle: false,
      result: '',
      show: false,
      time: 50 * 60, // in seconds
      timer: null,
    };
  },
  filters: {
    charIndex(i) {
      return String.fromCharCode(97 + i);
    },
  },
  methods: {
    restart() {
      this.chosenQuiz = false;
      this.questionIndex = 0;
      this.corrects = 0;
      this.incorrects = 0;
      this.userResponses = Array(this.quiz.questions.length).fill(null);
      this.time = 50 * 60;
    },
    chooseSubject(subject) {
      this.chosenQuiz = true;
      this.title = subject.title;
      this.quiz.questions = subject.jsonFile
        .sort(() => Math.random() - 0.5)
        .slice(0, subject.numQuestions);
      this.userResponses = Array(subject.numQuestions).fill(null);
      this.time = subject.time;
      this.corrects = 0;
      this.incorrects = 0;
      this.blank = 0;
      this.plusScore = subject.plusScore;
      this.minusScore = subject.minusScore;
    },
    selectOption(index, response) {
      if (this.userResponses[this.questionIndex] !== null
        && this.userResponses[this.questionIndex] !== index) {
        if (response.correct) {
          this.corrects += 1;
          this.incorrects -= 1;
        } else {
          this.corrects -= 1;
          this.incorrects += 1;
        }
      } else if (this.userResponses[this.questionIndex] !== index) {
        if (response.correct) {
          this.corrects += 1;
        } else {
          this.incorrects += 1;
        }
      }
      // eslint-disable-next-line no-undef
      Vue.set(this.userResponses, this.questionIndex, index);
      // console.log(this.userResponses);
    },
    next() {
      if (this.questionIndex < this.quiz.questions.length) {
        this.questionIndex += 1;
      }
    },

    prev() {
      if (this.questionIndex > 0) {
        this.questionIndex -= 1;
      }
    },
    // Return "true" count in userResponses
    score() {
      let score = 0;
      // eslint-disable-next-line no-plusplus
      for (let i = 0; i < this.userResponses.length; i++) {
        if (
          typeof this.quiz.questions[i].responses[
            this.userResponses[i]
          ] !== 'undefined'
          && this.quiz.questions[i].responses[this.userResponses[i]].correct
        ) {
          score += this.plusScore;
        }
        if (
          typeof this.quiz.questions[i].responses[
            this.userResponses[i]] !== 'undefined'
          && !this.quiz.questions[i].responses[this.userResponses[i]].correct
        ) {
          score -= this.minusScore;
        }
      }
      return Math.max(Math.round(score * 100) / 100, 0);

      // return this.userResponses.filter(function(val) { return val }).length;
    },
    decrementOrAlert() {
      if (this.time > 0) {
        // eslint-disable-next-line no-plusplus
        this.time--;
        return;
      }
      // alert('Se acabó el tiempo!');
      clearInterval(this.timer);
    },
  },
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  computed: {
    timeLeft() {
      return `${this.minutes}:${this.seconds}`;
    },
    minutes() {
      return String(Math.floor(this.time / 60)).padStart(2, '0');
    },
    seconds() {
      return String(this.time % 60).padStart(2, '0');
    },
  },
  created() {
    this.timer = setInterval(this.decrementOrAlert, 1000);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  @import url("https://fonts.googleapis.com/css?family=Montserrat:400,400i,700");
  @import url("https://fonts.googleapis.com/css?family=Open+Sans:400,400i,700");
  body {
    font-family: "Open Sans", sans-serif;
    font-size: 14px;
    height: 100vh;
    background: #cfd8dc;
    /* mocking native UI */
    cursor: default !important;
    /* remove text selection cursor */
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    /* remove text selection */
    user-drag: none;
    /* disbale element dragging */
    display: -webkit-box;
    display: flex;
    -webkit-box-align: center;
    align-items: center;
    -webkit-box-pack: center;
    justify-content: center;
  }

  .button {
    -webkit-transition: 0.3s;
    transition: 0.3s;
  }

  .title,
  .subtitle {
    font-family: Montserrat, sans-serif;
    font-weight: normal;
  }

  .animated {
    -webkit-transition-duration: 0.15s;
    transition-duration: 0.15s;
  }

  .container {
    margin: 0 0.5rem;
  }

  .questionBox {
    max-width: 30rem;
    width: 30rem;
    min-height: 30rem;
    background: #fafafa;
    position: relative;
    display: -webkit-box;
    display: flex;
    border-radius: 5px;
    overflow: hidden;
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.19), 0 6px 6px rgba(0, 0, 0, 0.23);
  }

  .questionBox header {
    background: rgba(0, 0, 0, 0.025);
    padding: 1.2rem;
    text-align: center;
    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  }

  .questionBox header h1 {
    font-weight: bold;
    margin-bottom: 1rem !important;
  }

  .questionBox header .progressContainer {
    width: 60%;
    margin: 0 auto;
  }
  .progressContainer {
    width: 60%;
    margin: 0 auto;
  }

  .questionBox header .progressContainer > progress {
    margin: 0;
    border-radius: 5rem;
    overflow: hidden;
    border: none;
    color: #3d5afe;
    width: 100%;
  }

  .questionBox header .progressContainer > progress::-moz-progress-bar {
    background: #3d5afe;
  }

  .questionBox header .progressContainer > progress::-webkit-progress-value {
    background: #3d5afe;
  }

  .questionBox header .progressContainer > p {
    margin: 0;
    margin-top: 0.5rem;
  }

  .questionBox .titleContainer {
    margin: 0 auto;
    padding: 1.5rem 1.5rem 0 1.5rem;
    text-align: left;
  }
  .questionBox .titleDate {
    padding: 0 1.5rem 0 1.5rem;
    text-align: left;
    font-style: italic;
    font-size: 0.8em;
    color: darkgrey;
  }

  .questionBox .quizForm {
    display: block;
    white-space: normal;
    height: 100%;
    width: 100%;
  }

  .questionBox .quizForm .quizFormContainer {
    height: 100%;
    margin: 15px 18px;
  }

  .questionBox .quizForm .quizFormContainer .field-label {
    text-align: left;
    margin-bottom: 0.5rem;
  }

  .questionBox .quizCompleted {
    width: 100%;
    padding: 1rem;
    text-align: center;
  }

  .questionBox .quizCompleted > .icon {
    color: #ff5252;
    font-size: 5rem;
  }

  .questionBox .quizCompleted > .icon .is-active {
    color: #00e676;
  }

  .questionBox .questionContainer {
    white-space: normal;
    height: 100%;
    width: 100%;
  }

  .questionBox .questionContainer .optionContainer {
    margin-top: 12px;
    -webkit-box-flex: 1;
    flex-grow: 1;
  }

  .questionBox .questionContainer .optionContainer .option {
    border-radius: 10px;
    padding: 9px 18px;
    margin: 0 18px;
    margin-bottom: 12px;
    -webkit-transition: 0.3s;
    transition: 0.3s;
    cursor: pointer;
    background-color: rgba(0, 0, 0, 0.05);
    color: rgba(0, 0, 0, 0.85);
    border: transparent 1px solid;
    text-align: left;
  }

  .questionBox .questionContainer .optionContainer .option.is-selected {
    border-color: rgba(0, 0, 0, 0.25);
    background-color: white;
  }

  .questionBox .questionContainer .optionContainer .option.is-correct {
    border-color: rgba(0, 0, 0, 0.25);
    background-color: rgba(186, 224, 120, 0.99);
  }

  .questionBox .questionContainer .optionContainer .option.is-wrong {
    border-color: rgba(0, 0, 0, 0.25);
    background-color: rgb(250, 140, 136);
  }

  /*.questionBox .questionContainer .optionContainer .option:hover {*/
  /*  background-color: rgba(0, 0, 0, 0.1);*/
  /*}*/

  .questionBox .questionContainer .optionContainer .option:active {
    -webkit-transform: scaleX(0.9);
    transform: scaleX(0.9);
  }

  .questionBox .questionContainer .questionFooter {
    background: rgba(0, 0, 0, 0.025);
    border-top: 1px solid rgba(0, 0, 0, 0.1);
    width: 100%;
    align-self: flex-end;
  }

  .questionBox .questionContainer .questionFooter .pagination {
    margin: 15px 25px;
  }

  .pagination {
    display: -webkit-box;
    display: flex;
    -webkit-box-pack: justify;
    justify-content: space-between;
  }

  .button {
    padding: 0.5rem 1rem;
    border: 1px solid rgba(0, 0, 0, 0.25);
    border-radius: 10px;
    margin: 0 0.25rem;
    -webkit-transition: 0.3s;
    transition: 0.3s;
  }

  .button:hover {
    cursor: pointer;
    background: #eceff1;
    border-color: rgba(0, 0, 0, 0.25);
  }
  .actionButton:hover {
    cursor: pointer;
    background: #6298c7;
    border-color: rgba(0, 0, 0, 0.25);
  }

  .button.is-active {
    background: #3d5afe;
    color: white;
    border-color: transparent;
  }

  .button.is-active:hover {
    background: #0a2ffe;
  }

  @media screen and (min-width: 769px) {
    .questionBox {
      -webkit-box-align: center;
      /*align-items: center;*/
      -webkit-box-pack: center;
      /*justify-content: center;*/
    }

    .questionBox .questionContainer {
      display: -webkit-box;
      display: flex;
      -webkit-box-orient: vertical;
      -webkit-box-direction: normal;
      flex-direction: column;
    }
  }
  @media screen and (max-width: 768px) {
    .sidebar {
      height: auto !important;
      border-radius: 6px 6px 0px 0px;
    }
  }


</style>
