<template>
  <div class="container">
    <h1>Memory-Spiel</h1>
    <section class="board">
      <Card
        v-for="(card, idx) in cards"
        :key="`card-${idx}`"
        :value="card.value"
        :visible="card.isShowing"
      />
    </section>
  </div>
</template>

<script>
import Card from "./components/Card.vue";

export default {
  name: "App",
  components: {
    Card,
  },
  setup() {
    const cards = [];

    for (let i = 0; i < 16; i++) {
      cards.push({
        value: i,
        isShowing: false,
      });
    }

    return {
      cards,
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
  grid-template-columns: 90px 90px 90px 90px;
  grid-template-rows: 90px 90px 90px 90px;
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  justify-content: center;
}
</style>
