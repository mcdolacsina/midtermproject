<template>
  <section class="final">
    <h2>{{ message }}</h2>
    <p v-html="result"></p>
    <div>
      <p class="answers-result">{{ correctAnswers }}/{{ numberOfQuestions }}</p>
      <div class="bar">
        <div
          :style="{ width: `${correctAnswersPercentage}%` }"
          class="correct"
        ></div>
        <div
          :style="{ width: `${wrongAnswersPercentage}%` }"
          class="wrong"
        ></div>
      </div>
    </div>

    <button @click="resetGame">Play again</button>
  </section>
</template>

<script>
import anime from "animejs/lib/anime.es.js";
export default {
  name: "Final",
  props: {
    correctAnswers: Number,
    numberOfQuestions: Number,
  },
  data() {
    return {
      message: "",
      result: "",
      correctAnswersPercentage: 0,
      wrongAnswersPercentage: 0,
    };
  },
  mounted() {
    this.calculatePercentages();
    this.changeResultText();

    anime({
      targets: ".correct",
      width: `${this.correctAnswersPercentage}%`,
      easing: "linear",
      loop: false,
      duration: 900,
    });

    anime({
      targets: ".wrong",
      width: `${this.wrongAnswersPercentage}%`,
      easing: "linear",
      loop: false,
      duration: 900,
    });
  },
  methods: {
    calculatePercentages() {
      this.correctAnswersPercentage =
        (this.correctAnswers / this.numberOfQuestions) * 100;
      this.wrongAnswersPercentage = 100 - this.correctAnswersPercentage;
    },
    changeResultText() {
      if (
        this.correctAnswersPercentage >= 0 &&
        this.correctAnswersPercentage <= 30
      ) {
        this.message = "Was it too hard?";
        this.result = `You only got ${this.correctAnswersPercentage}% of the questions right. You should try again...`;
      } else if (
        this.correctAnswersPercentage > 30 &&
        this.correctAnswersPercentage < 60
      ) {
        this.message = "Not so good...";
        this.result = `You got ${this.correctAnswersPercentage}% of the questions right. Maybe you can get a better score playing again!`;
      } else if (
        this.correctAnswersPercentage >= 60 &&
        this.correctAnswersPercentage < 80
      ) {
        this.message = "Well done!";
        this.result = `You got ${this.correctAnswersPercentage}% of the questions right. That's a good score! 🐢`;
      } else if (
        this.correctAnswersPercentage >= 80 &&
        this.correctAnswersPercentage < 95
      ) {
        this.message = "Very good!";
        this.result = `You got ${this.correctAnswersPercentage}% of the questions right! Congrats! 🐢`;
      } else if (this.correctAnswersPercentage >= 95) {
        this.message = "You're incredible!";
        this.result = `Exceptional! You got ${this.correctAnswersPercentage}% of the questions right! Congratulations! 🐢`;
      }
    },
    resetGame() {
      this.$emit("resetGame");
    },
  },
};
</script>

<style lang="scss" scoped>
@use "../App.scss";
.final {
  text-align: center;
  width: min(95%, 400px);
  @include App.flex();

  h2 {
    font-size: 2.5rem;
    color: App.$final-screen-heading;
  }

  p {
    font-size: 1.3rem;

    &.answers-result {
      color: #000;
      position: absolute;
      font-size: 1.2rem;
      left: 50%;
      top: 67%;
      height: 30px;
      transform: translate(-50%, -50%);
    }
  }

  > div {
    text-align: center;
    position: relative;
    width: 100%;

    .bar {
      background: rgb(40, 39, 49);
      box-shadow: 0px 0px 4px #2e2e4a;
      margin: 15px auto 0;
      width: 95%;
      height: 30px;
      border-radius: 5px;
      @include App.flex(row, space-between, stretch);
      .correct {
        border-radius: 5px 0 0 5px;
        background: App.$correct;
      }
      .wrong {
        border-radius: 0 5px 5px 0;
        background: App.$wrong;
      }
    }
  }
}
</style>