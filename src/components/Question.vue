<template>
  <section class="question">
    <h1>
      <span>{{ index + 1 }}</span
      >. <span v-html="questionText"></span>
    </h1>
    <div class="timer">
      <span class="clock"></span>
      <p>0:{{ secondsLeft }}</p>
    </div>

    <div class="answers">
      <div
        class="answer"
        @click="checkAnswer($event)"
        :style="[roundEnded ? '' : { background: '#fff' }]"
        v-html="question1"
      ></div>
      <div
        class="answer"
        @click="checkAnswer($event)"
        :style="[roundEnded ? '' : { background: '#fff' }]"
        v-html="question2"
      ></div>
      <div
        class="answer"
        @click="checkAnswer($event)"
        :style="[roundEnded ? '' : { background: '#fff' }]"
        v-html="question3"
      ></div>
      <div
        class="answer"
        @click="checkAnswer($event)"
        :style="[roundEnded ? '' : { background: '#fff' }]"
        v-html="question4"
      ></div>
    </div>

    <div
      class="post-answer"
      :style="[roundEnded ? { display: 'block' } : { display: 'none' }]"
    >
      <p class="message">
        {{ message }}
      </p>
      <button
        @click="goToNextQuestion"
        v-html="[
          index == numberOfQuestions - 1 ? 'See results!' : 'Next question',
        ]"
      ></button>
    </div>
  </section>
</template>

<script>
export default {
  name: "Question",
  props: {
    quizData: Object,
  },
  data() {
    return {
      gameFinished: false,
      correctAnswers: 0,
      secondsLeft: 0,
      index: 0,
      numberOfQuestions: 0,
      answers: [],
      questionText: "",
      question1: "",
      question2: "",
      question3: "",
      question4: "",
      userAnswered: false,
      roundEnded: false,
      isUserAnswerCorrect: undefined,
      message: "",
    };
  },
  mounted() {
    this.prepareQuestion();
  },

  methods: {
    prepareQuestion() {
      const questions = this.quizData.results;
      const index = this.index;

      this.numberOfQuestions = questions.length;

      // Put all the answers in an array to shuffle them later
      this.answers.push(questions[index].correct_answer);
      questions[index].incorrect_answers.forEach((answer) => {
        this.answers.push(answer);
      });

      this.correctAnswer = questions[index].correct_answer;

      this.questionText = questions[index].question;
      this.question1 = this.getRandomAnswer();
      this.question2 = this.getRandomAnswer();
      this.question3 = this.getRandomAnswer();
      this.question4 = this.getRandomAnswer();

      this.countdown();
    },

    getRandomAnswer() {
      const randomNumber = Math.floor(Math.random() * this.answers.length);
      let answer = this.answers.splice(randomNumber, 1)[0];

      return answer;
    },

    countdown() {
      this.secondsLeft = 45;
      this.timer = setInterval(() => {
        if (!this.userAnswered) {
          if (this.secondsLeft > 0) this.secondsLeft--;
          else if (this.secondsLeft <= 0 || this.roundEnded) this.stopTimer();
          if (this.secondsLeft.toString().length < 2)
            this.secondsLeft = `0${this.secondsLeft}`;
        }
        if (this.secondsLeft <= 0 && !this.userAnswered) {
          this.message = "Time ran out! No points in this round.";
          this.roundEnded = true;
        }
      }, 1000);
    },

    stopTimer() {
      clearInterval(this.timer);
    },

    checkAnswer(event) {
      if (!this.roundEnded) {
        this.userAnswered = true;
        this.roundEnded = true;

        this.isUserAnswerCorrect =
          event.target.innerText == this.correctAnswer ? true : false;

        this.message = this.isUserAnswerCorrect
          ? "Good job! 🐢"
          : "Better luck next time.";

        if (this.isUserAnswerCorrect) this.correctAnswers++;

        setTimeout(() => {
          event.target.style.background = this.isUserAnswerCorrect
            ? "#00e900"
            : "#ff4a4a";
        }, 200);
      }
    },

    goToNextQuestion() {
      this.checkIfItIsTheLastRound();

      if (!this.gameFinished) {
        this.index++;
        this.roundEnded = false;
        this.userAnswered = false;
        this.isUserAnswerCorrect = undefined;
        this.answers = [];
        this.stopTimer();
        this.prepareQuestion();
      }
    },

    checkIfItIsTheLastRound() {
      if (this.index == this.numberOfQuestions - 1) {
        this.gameFinished = true;
        this.$emit("endGame", {
          numberOfQuestions: this.numberOfQuestions,
          correctAnswers: this.correctAnswers,
        });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
@use "../App.scss";
.question {
  text-align: center;
  width: min(95%, 550px);

  h1 {
    width: 95%;
    margin: auto;
    text-align: justify;
  }

  .timer {
    @include App.flex(row);
    margin: 16px;

    .clock {
      display: inline-block;
      background: url(../assets/clock.svg) no-repeat center;
      height: 25px;
      margin-right: 5px;
      aspect-ratio: 1/1;
    }

    p {
      font-size: 1.3rem;
      color: App.$timer;
    }
  }
  .answers {
    display: grid;
    height: 180px;
    margin-bottom: 40px;
    gap: 10px;
    grid-template-columns: repeat(2, 1fr);
    grid-template-rows: repeat(2, 1fr);
    justify-items: center;

    .answer {
      font-size: 1.1rem;
      background: #fff;
      border-radius: 4px;
      transition: 0.3s;
      @include App.flex();
      color: black;
      width: 100%;
      cursor: default;
      box-shadow: 0px 0px 4px #2e2e4a;

      &:hover {
        transform: scale(1.03);
      }
    }
  }

  .post-answer {
    animation: fadeIn 1.5s;

    .message {
      font-size: 1.3rem;
      transition: 0.3s;
    }

    button {
      margin-top: 10px !important;
    }
  }
}

@media screen and (max-width: 600px) {
  .question {
    .answers {
      height: 250px;
      grid-template-columns: 1fr;
      grid-template-rows: repeat(4, 1fr);
    }
  }
}

@keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
</style>