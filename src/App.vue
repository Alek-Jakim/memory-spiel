<template>
  <div class="container">
    <h1>Memory-Spiel</h1>
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
      <h2>{{ currStatus }}</h2>
    </section>
  </div>
</template>

<script>
import { ref, watch } from "vue";
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
    const currStatus = ref("");

    for (let i = 0; i < 25; i++) {
      //I thought it was best to create an object instead of separately pushing the values
      //same as below, if I don't use .value, it give gives the following error: Must use `.value` to read or write the value wrapped by `ref()`
      cards.value.push({
        value: i,
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
            currStatus.value = "correct";

            cards.value[currVal[0].position].isCorrect = true;
            cards.value[currVal[1].position].isCorrect = true;
          } else {
            currStatus.value = "incorrect";

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

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100vh;
}

.board {
  margin-top: 50px;
  display: grid;
  grid-template-columns: 90px 90px 90px 90px 90px;
  grid-template-rows: 90px 90px 90px 90px 90px;
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  justify-content: center;
}
</style>
