<template>
  <div class="container">
    <h1>Memory-Spiel</h1>
    <h3 v-if="!isStarted" class="info">
      Click the "New Game" button to start/restart the game.
    </h3>
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
    const locked = new Audio("/sounds/locked.wav");
    const winGame = new Audio("/sounds/win.wav");

    //REACTIVE
    const cards = ref([]);
    const chosenCards = ref([]);
    const currStatus = computed(() => {
      if (cardPairsRemaining.value === 0) {
        winGame.play();
        return "You Won! Congratulations!";
      } else {
        return `Card Pairs - ${cardPairsRemaining.value}`;
      }
    });

    // function shuffleCards() {
    //   cards.value = _.shuffle(cards.value);
    // }

    //The names of the cards are based on the names of the image files
    //UPDATE: I decided to change the images
    const cardContent = [
      "tyrell",
      "stark",
      "white-walker",
      "baratheon",
      "dayne",
      "martell",
      "targaryen",
      "lanister",
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
      // Shuffles the cards when the game starts
      cards.value = _.shuffle(cards.value);

      //Plays the start sound, indicating that the game has begun
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

    //The name is self-explanatory, it changes the payload isisVisible to true
    const toggleFlipCard = (payload) => {
      if (
        !cards.value[payload.position].isStarted ||
        chosenCards.value.length === 2
      ) {
        locked.play();
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
        //Here we are watching the current value for any changes, namely the chosen cards
        if (currVal.length === 2) {
          const firstCard = currVal[0];
          const secondCard = currVal[1];

          if (firstCard.positionValue === secondCard.positionValue) {
            cards.value[firstCard.position].isCorrect = true;
            cards.value[secondCard.position].isCorrect = true;
            chosenCards.value.length = 0;
            correctAnswer.play();
          } else {
            //this makes it so that if you get match the wrong cards, the wrong card remains shortly revealed so the player has enough time to remember its position and try again
            wrongAnswer.play();
            setTimeout(() => {
              cards.value[firstCard.position].isVisible = false;
              cards.value[secondCard.position].isVisible = false;
              //this is implemented here so I can restrict the number of cards selected at 1.5 seconds, as well as the first if statement in the toggleFlipCard function
              chosenCards.value.length = 0;
            }, 1500);
          }
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
      locked,
      winGame,
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
  background-color: #1a0a2c;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 2000 1500'%3E%3Cdefs%3E%3Crect stroke='%231a0a2c' stroke-width='0.33' width='1' height='1' id='s'/%3E%3Cpattern id='a' width='3' height='3' patternUnits='userSpaceOnUse' patternTransform='rotate(31 1000 750) scale(35.65) translate(-971.95 -728.96)'%3E%3Cuse fill='%231f0f30' href='%23s' y='2'/%3E%3Cuse fill='%231f0f30' href='%23s' x='1' y='2'/%3E%3Cuse fill='%23231434' href='%23s' x='2' y='2'/%3E%3Cuse fill='%23231434' href='%23s'/%3E%3Cuse fill='%23281939' href='%23s' x='2'/%3E%3Cuse fill='%23281939' href='%23s' x='1' y='1'/%3E%3C/pattern%3E%3Cpattern id='b' width='7' height='11' patternUnits='userSpaceOnUse' patternTransform='rotate(31 1000 750) scale(35.65) translate(-971.95 -728.96)'%3E%3Cg fill='%232c1e3d'%3E%3Cuse href='%23s'/%3E%3Cuse href='%23s' y='5' /%3E%3Cuse href='%23s' x='1' y='10'/%3E%3Cuse href='%23s' x='2' y='1'/%3E%3Cuse href='%23s' x='2' y='4'/%3E%3Cuse href='%23s' x='3' y='8'/%3E%3Cuse href='%23s' x='4' y='3'/%3E%3Cuse href='%23s' x='4' y='7'/%3E%3Cuse href='%23s' x='5' y='2'/%3E%3Cuse href='%23s' x='5' y='6'/%3E%3Cuse href='%23s' x='6' y='9'/%3E%3C/g%3E%3C/pattern%3E%3Cpattern id='h' width='5' height='13' patternUnits='userSpaceOnUse' patternTransform='rotate(31 1000 750) scale(35.65) translate(-971.95 -728.96)'%3E%3Cg fill='%232c1e3d'%3E%3Cuse href='%23s' y='5'/%3E%3Cuse href='%23s' y='8'/%3E%3Cuse href='%23s' x='1' y='1'/%3E%3Cuse href='%23s' x='1' y='9'/%3E%3Cuse href='%23s' x='1' y='12'/%3E%3Cuse href='%23s' x='2'/%3E%3Cuse href='%23s' x='2' y='4'/%3E%3Cuse href='%23s' x='3' y='2'/%3E%3Cuse href='%23s' x='3' y='6'/%3E%3Cuse href='%23s' x='3' y='11'/%3E%3Cuse href='%23s' x='4' y='3'/%3E%3Cuse href='%23s' x='4' y='7'/%3E%3Cuse href='%23s' x='4' y='10'/%3E%3C/g%3E%3C/pattern%3E%3Cpattern id='c' width='17' height='13' patternUnits='userSpaceOnUse' patternTransform='rotate(31 1000 750) scale(35.65) translate(-971.95 -728.96)'%3E%3Cg fill='%23312341'%3E%3Cuse href='%23s' y='11'/%3E%3Cuse href='%23s' x='2' y='9'/%3E%3Cuse href='%23s' x='5' y='12'/%3E%3Cuse href='%23s' x='9' y='4'/%3E%3Cuse href='%23s' x='12' y='1'/%3E%3Cuse href='%23s' x='16' y='6'/%3E%3C/g%3E%3C/pattern%3E%3Cpattern id='d' width='19' height='17' patternUnits='userSpaceOnUse' patternTransform='rotate(31 1000 750) scale(35.65) translate(-971.95 -728.96)'%3E%3Cg fill='%231a0a2c'%3E%3Cuse href='%23s' y='9'/%3E%3Cuse href='%23s' x='16' y='5'/%3E%3Cuse href='%23s' x='14' y='2'/%3E%3Cuse href='%23s' x='11' y='11'/%3E%3Cuse href='%23s' x='6' y='14'/%3E%3C/g%3E%3Cg fill='%23352745'%3E%3Cuse href='%23s' x='3' y='13'/%3E%3Cuse href='%23s' x='9' y='7'/%3E%3Cuse href='%23s' x='13' y='10'/%3E%3Cuse href='%23s' x='15' y='4'/%3E%3Cuse href='%23s' x='18' y='1'/%3E%3C/g%3E%3C/pattern%3E%3Cpattern id='e' width='47' height='53' patternUnits='userSpaceOnUse' patternTransform='rotate(31 1000 750) scale(35.65) translate(-971.95 -728.96)'%3E%3Cg fill='%231c0d3b'%3E%3Cuse href='%23s' x='2' y='5'/%3E%3Cuse href='%23s' x='16' y='38'/%3E%3Cuse href='%23s' x='46' y='42'/%3E%3Cuse href='%23s' x='29' y='20'/%3E%3C/g%3E%3C/pattern%3E%3Cpattern id='f' width='59' height='71' patternUnits='userSpaceOnUse' patternTransform='rotate(31 1000 750) scale(35.65) translate(-971.95 -728.96)'%3E%3Cg fill='%231c0d3b'%3E%3Cuse href='%23s' x='33' y='13'/%3E%3Cuse href='%23s' x='27' y='54'/%3E%3Cuse href='%23s' x='55' y='55'/%3E%3C/g%3E%3C/pattern%3E%3Cpattern id='g' width='139' height='97' patternUnits='userSpaceOnUse' patternTransform='rotate(31 1000 750) scale(35.65) translate(-971.95 -728.96)'%3E%3Cg fill='%231c0d3b'%3E%3Cuse href='%23s' x='11' y='8'/%3E%3Cuse href='%23s' x='51' y='13'/%3E%3Cuse href='%23s' x='17' y='73'/%3E%3Cuse href='%23s' x='99' y='57'/%3E%3C/g%3E%3C/pattern%3E%3C/defs%3E%3Crect fill='url(%23a)' width='100%25' height='100%25'/%3E%3Crect fill='url(%23b)' width='100%25' height='100%25'/%3E%3Crect fill='url(%23h)' width='100%25' height='100%25'/%3E%3Crect fill='url(%23c)' width='100%25' height='100%25'/%3E%3Crect fill='url(%23d)' width='100%25' height='100%25'/%3E%3Crect fill='url(%23e)' width='100%25' height='100%25'/%3E%3Crect fill='url(%23f)' width='100%25' height='100%25'/%3E%3Crect fill='url(%23g)' width='100%25' height='100%25'/%3E%3C/svg%3E");
  background-attachment: fixed;
  background-size: cover;
  background-attachment: fixed;
  background-size: cover;
  background-attachment: fixed;
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
  transition: 0.3s ease-in-out;
}
button:hover {
  background: #7c6c8f;
  color: rgb(255, 255, 255);
  border: 2px solid #7c6c8f;
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
