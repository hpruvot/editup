<template>
  <div id="app" class="game">
    <div 
      v-for="(question, index) in questions" 
      :key="index"
      class="game__wrapper">
      <div class="game__zones">
        <div 
          v-for="(choice, index) in question.choices"
          :key="index" 
          :data-zone="index + 1"
          v-on:dragend="handleDragEnd"
          v-on:dragover="handleDragOver"
          v-on:drop="handleDrop"
          class="game__zone">
          <p>{{ choice }}</p>
        </div>
      </div>
    </div>
    <div class="game__drop">
      <div class="game__score"></div>
      <div class="game__center" 
        draggable="true"
        ref="gameCenter"
        v-on:dragend="handleDragEnd"
        v-on:dragstart="handleDragStart"
      >
        <div class="game__question">Pourquoi ?</div>
      </div>
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
      selectedDiv: null
    }
  },
  components: {},
  mounted () {
    axios
      .get('questions.json')
      .then(response => {
        this.questions = response.data.questions
      })
      .catch(error => {
        console.log(error)
        this.errored = true
      })
      .finally(() => this.loading = false)
  },
  methods: {
    handleDragEnd: function() {
      let selectedAnswer = this.selectedDiv.getAttribute('data-zone');

      if(this.selectedDiv) {
        anime({
          targets: '.game__center',
          scale: '1.5',
          translateX: '-50%',
          translateY: '-50%'
          // opacity: 0,
        });
      }

      if(selectedAnswer == this.questions[0].answer) {
        this.selectedDiv.classList.add('-correct');
      } else {
        this.selectedDiv.classList.add('-wrong');
      }
    },
    handleDragStart: function(e) {
      var crt = document.createElement('p');
      crt.style.display = "none"; /* or visibility: hidden, or any of the above */
      crt.style.opacity = "0"; /* or visibility: hidden, or any of the above */
      e.dataTransfer.setDragImage(crt, 0, 0);
    },
    handleDragOver: function(e) {
      e.preventDefault();
      document.querySelector('.game__center').style.pointerEvents = "none";

      anime({
        targets: '.game__center',
        left: e.x + 'px',
        top: e.y + 'px',
        duration: 500,
        elasticity: 100
      });
    },
    handleDrop: function(e) {
      this.selectedDiv = e.currentTarget;
    }
  }
}
</script>

<style lang="scss">
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
}

#app {
  width: 100%;
  height: 100%;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.game {
  &__wrapper {
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
      font-size: 30px;
      line-height: 1.2;
      margin: 0;
      transform-origin: 0 0;
      color: white;
      font-weight: bold;
      letter-spacing: 0.03em;
      text-shadow: 0px 2.76726px 2.21381px rgba(12, 13, 101, 0.0431739), 0px 6.6501px 5.32008px rgba(12, 13, 101, 0.0598892), 0px 12.5216px 10.0172px rgba(12, 13, 101, 0.0689504), 0px 22.3363px 17.869px rgba(12, 13, 101, 0.075418), 0px 41.7776px 33.4221px rgba(12, 13, 101, 0.0834553), 0px 100px 80px rgba(12, 13, 101, 0.11);
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
      background: greenyellow;
    }

    &.-wrong {
      background: crimson;
    }
  }

  &__score {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    transform-origin: 0% 0%;
    width: 300px;
    height: 300px;
    border-radius: 100%;
    background: grey;
    cursor: pointer;
  }

  &__center {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    transform-origin: 0% 0%;
    width: 200px;
    height: 200px;
    border-radius: 100%;
    background: white;
    cursor: pointer;
  }

  &__question {
    font-weight: bold;
    border-bottom: 2px solid #FF0077;
  }
}
</style>
