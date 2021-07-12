<template>
  <div class="container">
    <h1>Memory-Spiel</h1>
    <h3 class="info">Click the "New Game" button to start/restart the game.</h3>
    <section class="board">
      <!--this is the card component and I'm looping through to generate 16 cards, while passing the props, similar to react-->
      <Card
        v-for="(card, idx) in cards"
        :key="`card-${idx}`"
        :value="card.value"
        :position="card.position"
        :isVisible="card.isVisible"
        :isCorrect="card.isCorrect"
        :isStarted="card.isStarted"
        @select-card="toggleFlipCard"
      />
    </section>
    <section class="board-ctrl">
      <h3>{{ currStatus }}</h3>
      <button @click="restartGame">New Game</button>
    </section>
  </div>
</template>

<script>
import _ from "lodash";
import { computed, ref, watch } from "vue";
import Card from "./components/Card.vue";

export default {
  name: "App",
  components: {
    Card,
  },
  setup() {
    //wrap the array with the ref method because this is not "reactive" as with react/svelte, this is just regular JS

    //SOUNDS
    const startSound = new Audio("/sounds/start.wav");
    const correctAnswer = new Audio("/sounds/correct.wav");
    const wrongAnswer = new Audio("/sounds/wrong.wav");

    //REACTIVE
    const cards = ref([]);
    const chosenCards = ref([]);
    const currStatus = computed(() => {
      if (cardPairsRemaining.value === 0) {
        return "You Won! Congratulations!";
      } else {
        return `Card Pairs - ${cardPairsRemaining.value}`;
      }
    });

    // function shuffleCards() {
    //   cards.value = _.shuffle(cards.value);
    // }

    //the different values/pictures used
    const cardContent = [
      "octopus",
      "shark",
      "starfish",
      "turtle",
      "shell",
      "crab",
      "seahorse",
      "monster",
    ];

    cardContent.forEach((element) => {
      cards.value.push({
        value: element,
        isVisible: false,
        position: null,
        isCorrect: false,
        isStarted: false,
      });

      cards.value.push({
        value: element,
        isVisible: false,
        position: null,
        isCorrect: false,
        isStarted: false,
      });
    });

    // Updates and aligns the position with the index
    cards.value = cards.value.map((card, index) => {
      return {
        ...card,
        position: index,
      };
    });

    //re-evaluates the remaining card pairs
    const cardPairsRemaining = computed(() => {
      const cardsRemaining = cards.value.filter(
        (card) => card.isCorrect === false
      ).length;

      return cardsRemaining / 2;
    });

    const restartGame = () => {
      cards.value = _.shuffle(cards.value);

      startSound.play();

      cards.value = cards.value.map((card, index) => {
        return {
          ...card,
          isCorrect: false,
          isVisible: false,
          position: index,
          isStarted: true,
        };
      });
    };

    // for (let i = 0; i < 16; i++) {
    //I thought it was best to create an object instead of separately pushing the values
    //same as below, if I don't use .value, it give gives the following error: Must use `.value` to read or write the value wrapped by `ref()`
    //   cards.value.push({
    //     value: 8,
    //     isVisible: false,
    //     position: i,
    //     isCorrect: false,
    //   });
    // }

    //The name is self-explanatory, it changes the payload isisVisible to true
    const toggleFlipCard = (payload) => {
      if (!cards.value[payload.position].isStarted) {
        return;
      } else {
        //you need to unpack the cards array with .value otherwise it gives you an error
        cards.value[payload.position].isVisible = true;
        //!cards.value[payload.position].isVisible - if you wanna switch on/off;

        if (chosenCards.value[0]) {
          if (
            chosenCards.value[0].position === payload.position &&
            chosenCards.value[0].positionValue === payload.positionValue
          ) {
            return;
          } else {
            chosenCards.value[1] = payload;
          }
        } else {
          chosenCards.value[0] = payload;
        }
      }
    };

    watch(
      chosenCards,
      (currVal) => {
        //currVal stands for currentValue, I just prefer shorter names for arguments

        if (currVal.length === 2) {
          const firstCard = currVal[0];
          const secondCard = currVal[1];

          if (firstCard.positionValue === secondCard.positionValue) {
            cards.value[firstCard.position].isCorrect = true;
            cards.value[secondCard.position].isCorrect = true;
            correctAnswer.play();
          } else {
            //this makes it so that if you get match the wrong cards, the wrong card remains shortly revealed so the player has enough time to remember its position and try again
            wrongAnswer.play();
            setTimeout(() => {
              cards.value.isStarted = false;
              cards.value[firstCard.position].isVisible = false;
              cards.value[secondCard.position].isVisible = false;
              document.querySelector(".card").disabled = true;
            }, 2000);
          }

          // currVal[0].positionValue === currVal[1].positionValue
          //   ? (currStatus.value = "correct")
          //   : (currStatus.value = "incorrect");

          chosenCards.value.length = 0;
        }
      },

      //you need to add deep: true to track the values in the array, otherwise nothing happens
      { deep: true }
    );

    return {
      cards,
      toggleFlipCard,
      chosenCards,
      currStatus,
      restartGame,
      startSound,
      correctAnswer,
      wrongAnswer,
    };
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Source+Code+Pro&display=swap");

* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}

body {
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' version='1.1' xmlns:xlink='http://www.w3.org/1999/xlink' xmlns:svgjs='http://svgjs.com/svgjs' width='1440' height='560' preserveAspectRatio='none' viewBox='0 0 1440 560'%3e%3cg mask='url(%26quot%3b%23SvgjsMask1000%26quot%3b)' fill='none'%3e%3crect width='1440' height='560' x='0' y='0' fill='%230e2a47'%3e%3c/rect%3e%3cpath d='M 0%2c136 C 57.6%2c148.8 172.8%2c211.4 288%2c200 C 403.2%2c188.6 460.8%2c90 576%2c79 C 691.2%2c68 748.8%2c150.4 864%2c145 C 979.2%2c139.6 1036.8%2c28.2 1152%2c52 C 1267.2%2c75.8 1382.4%2c221.6 1440%2c264L1440 560L0 560z' fill='%23184a7e'%3e%3c/path%3e%3cpath d='M 0%2c280 C 96%2c334.8 288%2c545.6 480%2c554 C 672%2c562.4 768%2c346.4 960%2c322 C 1152%2c297.6 1344%2c410 1440%2c432L1440 560L0 560z' fill='%232264ab'%3e%3c/path%3e%3c/g%3e%3cdefs%3e%3cmask id='SvgjsMask1000'%3e%3crect width='1440' height='560' fill='white'%3e%3c/rect%3e%3c/mask%3e%3c/defs%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-size: cover;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

#app {
  font-family: "Source Code Pro", monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #fff;
  margin-top: 20px;
}

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.board {
  margin: 20px auto;
  width: 100%;
  display: grid;
  grid-template-columns: repeat(4, 90px);
  grid-template-rows: repeat(4, 90px);
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  justify-content: center;
}

/*
.btn-container {
  width: 400px;
  margin: 30px auto;
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  flex-direction: row;
}
*/

.info {
  margin: 20px auto;
}

button {
  padding: 5px 10px;
  margin: 20px;
  font-size: 1.1rem;
  border: 2px solid #000;
  border-radius: 5px;

  outline: none;
  background: #fff;
  color: rgb(36, 34, 129);
  transition: 0.5s ease-in-out;
}
button:hover {
  background: rgb(36, 34, 129);
  color: rgb(255, 255, 255);
  border: 2px solid rgb(36, 34, 129);
  border-radius: 10px;
}

button:active {
  transform: scale(0.7);
}

img {
  max-height: 100%;
  max-width: 100%;
}
.board-ctrl {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  max-height: 100%;
}
</style>
