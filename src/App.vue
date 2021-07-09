<template>
  <div class="container">
    <h1>Memory-Spiel</h1>
    <h3>{{ currStatus }}</h3>
    <section class="board">
      <!--this is the card component and I'm looping through to generate 16 cards, while passing the props, similar to react-->
      <Card
        v-for="(card, idx) in cards"
        :key="`card-${idx}`"
        :value="card.value"
        :position="card.position"
        :isVisible="card.isVisible"
        :isCorrect="card.isCorrect"
        @select-card="toggleFlipCard"
      />

      <!--<h2>{{ chosenCards }}</h2>-->
    </section>
    <button @click="restartGame">Restart</button>
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
    //REACTIVE
    const cards = ref([]);
    const chosenCards = ref([]);
    const currStatus = computed(() => {
      if (cardPairsRemaining.value === 0) {
        return "You Won! Congratulations!";
      } else {
        return `Card Pairs Remaining - ${cardPairsRemaining.value}`;
      }
    });

    const cardContent = [1, 2, 3, 4, 5, 6, 7, 8];

    cardContent.forEach((element) => {
      cards.value.push({
        value: element,
        isVisible: false,
        position: null,
        isCorrect: false,
      });

      cards.value.push({
        value: element,
        isVisible: false,
        position: null,
        isCorrect: false,
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

    const shuffleCards = () => {
      cards.value = _.shuffle(cards.value);
    };

    const restartGame = () => {
      shuffleCards();

      cards.value = cards.value.map((card, index) => {
        return {
          ...card,
          isCorrect: false,
          isVisible: false,
          position: index,
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
      //you need to unpack the cards array with .value otherwise it gives you an error
      cards.value[payload.position].isVisible = true;
      //!cards.value[payload.position].isVisible - if you wanna switch on/off;

      if (chosenCards.value[0]) {
        //If I clicked on the same card twice, it registered it as true so this if statement is to prevent that
        if (
          chosenCards.value[0].position === payload.position &&
          chosenCards.value[0].positionValue === payload.positionValue
        ) {
          return;
        }

        chosenCards.value[1] = payload;
      } else {
        chosenCards.value[0] = payload;
      }

      // chosenCards.value[0]
      //   ? (chosenCards.value[1] = payload)
      //   : (chosenCards.value[0] = payload);
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
          } else {
            //this makes it so that if you get match the wrong cards, the wrong card remains shortly revealed so the player has enough time to remember its position and try again
            setTimeout(() => {
              cards.value[firstCard.position].isVisible = false;
              cards.value[secondCard.position].isVisible = false;
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
      shuffleCards,
      restartGame,
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
  background-color: #250943;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='40' height='40' viewBox='0 0 100 100'%3E%3Crect x='0' y='0' width='46' height='46' fill-opacity='0.6' fill='%23280464'/%3E%3C/svg%3E");
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
  margin-top: 60px;
}

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.board {
  margin: 50px auto;
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
button {
  padding: 15px;
  margin-bottom: 40px;
  font-size: 1.1rem;
  border: none;
  outline: none;
  background: rgb(110, 51, 110);
  color: #fff;
  transition: 0.5s ease-in-out;
}
button:hover {
  background: #fff;
  color: rgb(110, 51, 110);
  border-radius: 10px;
}
</style>
