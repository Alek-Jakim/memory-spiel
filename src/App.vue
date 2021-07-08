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
        :visible="card.visible"
        @select-card="flipCard"
      />
    </section>
  </div>
</template>

<script>
import { ref } from "vue";
import Card from "./components/Card.vue";

export default {
  name: "App",
  components: {
    Card,
  },
  setup() {
    //wrap the array with the ref method because this is not "reactive" as with react/svelte, this is just regular JS
    const cards = ref([]);

    for (let i = 0; i < 25; i++) {
      //I thought it was best to create an object instead of separately pushing the values
      //same as below, if I don't use .value, it give gives the following error: Must use `.value` to read or write the value wrapped by `ref()`
      cards.value.push({
        value: i,
        visible: false,
        position: i,
      });
    }

    //The name is self-explanatory, it changes the payload isVisible to true
    const flipCard = (payload) => {
      //you need to unpack the cards array with .value otherwise it gives you an error
      cards.value[payload.position].visible =
        !cards.value[payload.position].visible;
    };

    return {
      cards,
      flipCard,
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
