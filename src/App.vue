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
      <button @click="shuffleCards">Shuffle Cards</button>
      <button @click="restartGame">Restart</button>
      <!--<h2>{{ chosenCards }}</h2>-->
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

    //
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

    for (let i = 0; i < 16; i++) {
      //I thought it was best to create an object instead of separately pushing the values
      //same as below, if I don't use .value, it give gives the following error: Must use `.value` to read or write the value wrapped by `ref()`
      cards.value.push({
        value: 8,
        isVisible: false,
        position: i,
        isCorrect: false,
      });
    }

    //The name is self-explanatory, it changes the payload isisVisible to true
    const toggleFlipCard = (payload) => {
      //you need to unpack the cards array with .value otherwise it gives you an error
      cards.value[payload.position].isVisible = true;
      //!cards.value[payload.position].isVisible - if you wanna switch on/off;

      chosenCards.value[0]
        ? (chosenCards.value[1] = payload)
        : (chosenCards.value[0] = payload);
    };

    watch(
      chosenCards,
      (currVal) => {
        //currVal stands for currentValue, I just prefer shorter names for arguments

        if (currVal.length === 2) {
          if (currVal[0].positionValue === currVal[1].positionValue) {
            cards.value[currVal[0].position].isCorrect = true;
            cards.value[currVal[1].position].isCorrect = true;
          } else {
            cards.value[currVal[0].position].isVisible = false;
            cards.value[currVal[1].position].isVisible = false;
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
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #a1cdf8;
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
  display: grid;
  grid-template-columns: 90px 90px 90px 90px;
  grid-template-rows: 90px 90px 90px 90px;
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  justify-content: center;
}
</style>
