<template>
  <div id="app" class="game">
    <div class="game__play"
      v-on:dragend="handleDragEnd"
      v-on:dragover="handleDragOver"
      v-on:drop="handleDrop"
      v-on:touchend="handleDrop"
      v-on:touchmove="handleDragOver"
      v-show="!endGame">
      <div class="game__score">
        {{ scoring.left }}
        {{ scoring.right }}
      </div>
      <div 
        v-for="(question, index) in questions" 
        :key="index"
        v-show="index === questionIndex"
        class="game__wrapper">
        <div class="game__zones">
          <div 
            v-for="(choice, index) in question.choices"
            :key="index" 
            :data-zone="index + 1"
            class="game__zone">
            <p>{{ choice }}</p>
          </div>
        </div>
      </div>
      <div class="game__drop">
        <div class="game__scoring"></div>
        <div class="game__center" 
          draggable="true"
          ref="gameCenter"
          v-on:dragend="handleDragEnd"
          v-on:dragstart="handleDragStart"
        >
          <div class="game__question">{{ currentQuestion }}</div>
        </div>
      </div>
    </div>
    <div class="game__results" v-show="endGame">
      Results !
    </div>
  </div>
</template>

<script>
import anime from 'animejs/lib/anime.es.js';
import axios from 'axios';

export default {
  name: 'App',
  data () {
    return {
      questions: null,
      loading: true,
      errored: false,
      selectedDiv: null,
      currentQuestion: null,
      endGame: false,
      // Store current question index
      questionIndex: 0,
      // Positions
      positions: {
        elementInitialTop: null,
        elementInitialLeft: null,
        clientX: undefined,
        clientY: undefined,
        movementX: 0,
        movementY: 0
      },
      scoring: {
        left: 0,
        right: 0
      }
    }
  },
  components: {},
  created() {
    window.addEventListener("resize", this.handleResize);
  },
  mounted () {
    this.positions.elementInitialTop = this.$refs.gameCenter.offsetTop + 'px';
    this.positions.elementInitialLeft = this.$refs.gameCenter.offsetLeft + 'px';
    axios
      .get('questions.json')
      .then(response => {
        this.questions = response.data.questions
        this.currentQuestion = response.data.questions[0].question
      })
      .catch(error => {
        console.log(error)
        this.errored = true
      })
      .finally(() => this.loading = false)
  },
  methods: {
    handleResize: function() {
      console.log("handleResize");
      if(this.$refs.gameCenter) {
        this.$refs.gameCenter.style.top = "";
        this.$refs.gameCenter.style.left = "";
        this.positions.elementInitialTop = this.$refs.gameCenter.offsetTop + 'px';
        this.positions.elementInitialLeft = this.$refs.gameCenter.offsetLeft + 'px';
      }
    },
    handleDragStart: function(e) {
      console.log("handleDragStart")
      var crt = document.createElement('p');
      crt.style.display = "none"; /* or visibility: hidden, or any of the above */
      crt.style.opacity = "0"; /* or visibility: hidden, or any of the above */
      if(e.dataTransfer) {
        e.dataTransfer.setDragImage(crt, 0, 0);
      }
    },
    handleDragEnd: function(e) {
      console.log("handleDragEnd")
      let animationCompleted = false;

      if(this.selectedDiv && !animationCompleted) {
        console.log("Let's animate !", this.selectedDiv)
        e.preventDefault();
        e.stopPropagation();
        let selectedAnswer = this.selectedDiv.getAttribute('data-zone');

        if(selectedAnswer == this.questions[this.questionIndex].answer) {
          this.selectedDiv.classList.add('-correct');
          this.correctAnswer(selectedAnswer);
        } else {
          this.selectedDiv.classList.add('-wrong');
          this.wrongAnswer();
        }
      } else {
        this.wrongAnswer();
      }
    },
    handleDragOver: function(event) {
      console.log("handleDragOver")
      event.preventDefault();
      document.querySelector('.game__center').style.pointerEvents = "none";

      this.positions.movementX = event.clientX ? this.positions.clientX - event.clientX : this.positions.clientX - event.touches[0].clientX
      this.positions.movementY = event.clientY ? this.positions.clientY - event.clientY : this.positions.clientY - event.touches[0].clientY
      this.positions.clientX = event.clientX ? event.clientX : event.touches[0].clientX
      this.positions.clientY = event.clientY ? event.clientY : event.touches[0].clientY
      // set the element's new position:
      this.$refs.gameCenter.style.top = (this.$refs.gameCenter.offsetTop - this.positions.movementY) + 'px'
      this.$refs.gameCenter.style.left = (this.$refs.gameCenter.offsetLeft - this.positions.movementX) + 'px'
    },
    handleDrop: function(e) {
      console.log("handleDrop", e)
      let zone = e.type == "touchend" ? document.elementFromPoint(e.changedTouches[0].clientX, e.changedTouches[0].clientY) : e.target
      
      if(zone.classList.contains('game__zone')) {
        this.selectedDiv = zone;
        this.handleDragEnd(e);
      } else {
        this.resetCenter();
      }
    },
    correctAnswer: function(selectedAnswer) {
      console.log("correctAnswer")
      this.endGame = this.questionIndex + 1 == this.questions.length;

      if(!this.endGame) {
        selectedAnswer == 1 ? this.scoring.left++ : this.scoring.right++
        anime({
          targets: '.game__center',
          opacity: 0,
          duration: 500, // Can be inherited
          complete: function() {
            this.questionIndex++
            this.$refs.gameCenter.style.top = this.positions.elementInitialTop
            this.$refs.gameCenter.style.left = this.positions.elementInitialLeft
            this.$refs.gameCenter.style.opacity = 1
            this.currentQuestion = this.questions[this.questionIndex].question
          }.bind(this)
        });

        this.resetCenter();
      }
    },
    wrongAnswer: function() {
      console.log("wrongAnswer")
      anime({
        targets: '.game__center',
        top: [this.$refs.gameCenter.style.top, this.positions.elementInitialTop], // from 100 to 250
        left: [this.$refs.gameCenter.style.left, this.positions.elementInitialLeft], // from 100 to 250
        duration: 500,
        opacity: 1,
        elasticity: 50
      });
      this.resetCenter();
    },
    resetCenter: function() {
      console.log("reset center")
      this.positions.clientX = undefined
      this.positions.clientY = undefined
      this.positions.movementX = 0
      this.positions.movementY = 0
      document.querySelector('.game__center').style.pointerEvents = "";
    },
    handleHoverScore(e) {
      console.log(e.target);
      this.currentQuestion = this.questions[this.questionIndex].question
    }
  }
}
</script>

<style lang="scss">
html,
body {
  margin: 0;
  padding: 0;
  height: 100vh;
  overflow: hidden;
}

#app {
  width: 100%;
  height: 100%;
  overflow: hidden;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.game {
  &__wrapper {
    position: absolute;
    width: 100%;
    height: 100%;
    user-select: none;
  }

  &__zones {
    display: flex;
    width: 100%;
    height: 100%;
  }

  &__zone {
    position: relative;
    width: 50%;
    height: 100%;
    transition: background 0.3s;

    p {
      position: absolute;
      top: 50%;
      left: 50%;
      font-size: 20px;
      line-height: 1.2;
      margin: 0;
      transform-origin: 0 0;
      color: white;
      font-weight: bold;
      white-space: nowrap;
      letter-spacing: 0.03em;
      text-shadow: 0px 2.76726px 2.21381px rgba(12, 13, 101, 0.0431739), 0px 6.6501px 5.32008px rgba(12, 13, 101, 0.0598892), 0px 12.5216px 10.0172px rgba(12, 13, 101, 0.0689504), 0px 22.3363px 17.869px rgba(12, 13, 101, 0.075418), 0px 41.7776px 33.4221px rgba(12, 13, 101, 0.0834553), 0px 100px 80px rgba(12, 13, 101, 0.11);

      @media screen and (min-width: 786px) {
        font-size: 30px;
      }
    }

    &:nth-of-type(1) {
      background: #ECD078;

      p {
        transform: rotate(-90deg) translate(-50%, -50%);
      }
    }

    &:nth-of-type(2) {
      background: #D95B43;

      p {
        transform: rotate(90deg) translate(-50%, -50%);
      }
    }

    &:nth-of-type(3) {
      background: #C02942;

      p {
        transform: rotate(90deg) translate(-50%, -50%);
      }
    }

    &:nth-of-type(4) {
      background: #542437;

      p {
        transform: rotate(90deg) translate(-50%, -50%);
      }
    }

    &:nth-of-type(5) {
      background: #53777A;

      p {
        transform: rotate(90deg) translate(-50%, -50%);
      }
    }

    &:nth-of-type(6) {
      background: #8A9B0F;

      p {
        transform: rotate(90deg) translate(-50%, -50%);
      }
    }

    &.-correct {
      animation: rightAnswer 500ms linear forwards alternate;
    }

    &.-wrong {
      animation: wrongAnswer 500ms linear forwards alternate;
    }
  }

  &__scoring {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    transform-origin: 0% 0%;
    width: 150px;
    height: 150px;
    border-radius: 100%;
    background: grey;
    cursor: pointer;

    @media screen and (min-width: 786px) {
      width: 300px;
      height: 300px;
    }
  }

  &__center {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    top: 373px;
    left: 604px;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    transform-origin: 0% 0%;
    width: 100px;
    height: 100px;
    border-radius: 100%;
    background: white;
    cursor: pointer;

    @media screen and (min-width: 786px) {
      width: 200px;
      height: 200px;
    }
  }

  &__question {
    font-weight: bold;
    padding: 5px;
    font-size: 14px;

    @media screen and (min-width: 786px) {
      font-size: 16px;
    }
  }
}

@keyframes wrongAnswer {
  0% {}
  50% {
    background: crimson
  }
  100% {}
}

@keyframes rightAnswer {
  0% {}
  50% {
    background: greenyellow
  }
  100% {}
}
</style>
