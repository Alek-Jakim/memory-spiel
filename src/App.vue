<template>
  <div class="container">
    <h1>Memory-Spiel</h1>
    <h3 v-if="!isStarted" class="info">
      {{ currStatus }}
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
        return "Congratulations, you won!";
      } else if (
        cardPairsRemaining.value > 0 &&
        cards.value[cards.value.length - 1].isStarted
      ) {
        return `Game Started! Pairs Remaining - ${cardPairsRemaining.value}`;
      } else {
        return 'Click "New Game" to start or restart the game.';
      }
    });

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

      console.log(cards.value[0].isStarted);

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
      // console.log(cards.value[payload.position]);

      //this if statement makes sure that you can't click on cards already guessed, it was quite an annoying bug
      if (cards.value[payload.position].isCorrect) {
        return;
      } else {
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
            //If the position values match, it means it's the cards match
            cards.value[firstCard.position].isCorrect = true;
            cards.value[secondCard.position].isCorrect = true;
            //Reset the length so that only 2 cards can be clicked at one time
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
            }, 2000);
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
      cardPairsRemaining,
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
  background-color: #c2baff;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='100%25'%3E%3Cdefs%3E%3ClinearGradient id='a' gradientUnits='userSpaceOnUse' x1='0' x2='0' y1='0' y2='100%25' gradientTransform='rotate(240)'%3E%3Cstop offset='0' stop-color='%23c2baff'/%3E%3Cstop offset='1' stop-color='%233245ff'/%3E%3C/linearGradient%3E%3Cpattern patternUnits='userSpaceOnUse' id='b' width='540' height='450' x='0' y='0' viewBox='0 0 1080 900'%3E%3Cg fill-opacity='0.1'%3E%3Cpolygon fill='%23444' points='90 150 0 300 180 300'/%3E%3Cpolygon points='90 150 180 0 0 0'/%3E%3Cpolygon fill='%23AAA' points='270 150 360 0 180 0'/%3E%3Cpolygon fill='%23DDD' points='450 150 360 300 540 300'/%3E%3Cpolygon fill='%23999' points='450 150 540 0 360 0'/%3E%3Cpolygon points='630 150 540 300 720 300'/%3E%3Cpolygon fill='%23DDD' points='630 150 720 0 540 0'/%3E%3Cpolygon fill='%23444' points='810 150 720 300 900 300'/%3E%3Cpolygon fill='%23FFF' points='810 150 900 0 720 0'/%3E%3Cpolygon fill='%23DDD' points='990 150 900 300 1080 300'/%3E%3Cpolygon fill='%23444' points='990 150 1080 0 900 0'/%3E%3Cpolygon fill='%23DDD' points='90 450 0 600 180 600'/%3E%3Cpolygon points='90 450 180 300 0 300'/%3E%3Cpolygon fill='%23666' points='270 450 180 600 360 600'/%3E%3Cpolygon fill='%23AAA' points='270 450 360 300 180 300'/%3E%3Cpolygon fill='%23DDD' points='450 450 360 600 540 600'/%3E%3Cpolygon fill='%23999' points='450 450 540 300 360 300'/%3E%3Cpolygon fill='%23999' points='630 450 540 600 720 600'/%3E%3Cpolygon fill='%23FFF' points='630 450 720 300 540 300'/%3E%3Cpolygon points='810 450 720 600 900 600'/%3E%3Cpolygon fill='%23DDD' points='810 450 900 300 720 300'/%3E%3Cpolygon fill='%23AAA' points='990 450 900 600 1080 600'/%3E%3Cpolygon fill='%23444' points='990 450 1080 300 900 300'/%3E%3Cpolygon fill='%23222' points='90 750 0 900 180 900'/%3E%3Cpolygon points='270 750 180 900 360 900'/%3E%3Cpolygon fill='%23DDD' points='270 750 360 600 180 600'/%3E%3Cpolygon points='450 750 540 600 360 600'/%3E%3Cpolygon points='630 750 540 900 720 900'/%3E%3Cpolygon fill='%23444' points='630 750 720 600 540 600'/%3E%3Cpolygon fill='%23AAA' points='810 750 720 900 900 900'/%3E%3Cpolygon fill='%23666' points='810 750 900 600 720 600'/%3E%3Cpolygon fill='%23999' points='990 750 900 900 1080 900'/%3E%3Cpolygon fill='%23999' points='180 0 90 150 270 150'/%3E%3Cpolygon fill='%23444' points='360 0 270 150 450 150'/%3E%3Cpolygon fill='%23FFF' points='540 0 450 150 630 150'/%3E%3Cpolygon points='900 0 810 150 990 150'/%3E%3Cpolygon fill='%23222' points='0 300 -90 450 90 450'/%3E%3Cpolygon fill='%23FFF' points='0 300 90 150 -90 150'/%3E%3Cpolygon fill='%23FFF' points='180 300 90 450 270 450'/%3E%3Cpolygon fill='%23666' points='180 300 270 150 90 150'/%3E%3Cpolygon fill='%23222' points='360 300 270 450 450 450'/%3E%3Cpolygon fill='%23FFF' points='360 300 450 150 270 150'/%3E%3Cpolygon fill='%23444' points='540 300 450 450 630 450'/%3E%3Cpolygon fill='%23222' points='540 300 630 150 450 150'/%3E%3Cpolygon fill='%23AAA' points='720 300 630 450 810 450'/%3E%3Cpolygon fill='%23666' points='720 300 810 150 630 150'/%3E%3Cpolygon fill='%23FFF' points='900 300 810 450 990 450'/%3E%3Cpolygon fill='%23999' points='900 300 990 150 810 150'/%3E%3Cpolygon points='0 600 -90 750 90 750'/%3E%3Cpolygon fill='%23666' points='0 600 90 450 -90 450'/%3E%3Cpolygon fill='%23AAA' points='180 600 90 750 270 750'/%3E%3Cpolygon fill='%23444' points='180 600 270 450 90 450'/%3E%3Cpolygon fill='%23444' points='360 600 270 750 450 750'/%3E%3Cpolygon fill='%23999' points='360 600 450 450 270 450'/%3E%3Cpolygon fill='%23666' points='540 600 630 450 450 450'/%3E%3Cpolygon fill='%23222' points='720 600 630 750 810 750'/%3E%3Cpolygon fill='%23FFF' points='900 600 810 750 990 750'/%3E%3Cpolygon fill='%23222' points='900 600 990 450 810 450'/%3E%3Cpolygon fill='%23DDD' points='0 900 90 750 -90 750'/%3E%3Cpolygon fill='%23444' points='180 900 270 750 90 750'/%3E%3Cpolygon fill='%23FFF' points='360 900 450 750 270 750'/%3E%3Cpolygon fill='%23AAA' points='540 900 630 750 450 750'/%3E%3Cpolygon fill='%23FFF' points='720 900 810 750 630 750'/%3E%3Cpolygon fill='%23222' points='900 900 990 750 810 750'/%3E%3Cpolygon fill='%23222' points='1080 300 990 450 1170 450'/%3E%3Cpolygon fill='%23FFF' points='1080 300 1170 150 990 150'/%3E%3Cpolygon points='1080 600 990 750 1170 750'/%3E%3Cpolygon fill='%23666' points='1080 600 1170 450 990 450'/%3E%3Cpolygon fill='%23DDD' points='1080 900 1170 750 990 750'/%3E%3C/g%3E%3C/pattern%3E%3C/defs%3E%3Crect x='0' y='0' fill='url(%23a)' width='100%25' height='100%25'/%3E%3Crect x='0' y='0' fill='url(%23b)' width='100%25' height='100%25'/%3E%3C/svg%3E");
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
  width: 100%;
  font-size: 1.1rem;
  border: 2px solid #000;
  border-radius: 5px;
  outline: none;
  background: #fff;
  color: rgb(36, 34, 129);
  transition: 0.3s ease-in-out;
}
button:hover {
  background: #0f4192;
  color: rgb(255, 255, 255);
  border: 2px solid #0f4192;
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
