<template>
  <div class="container">
    <h1>Memory-Spiel</h1>
    <section class="spielbrett">
      <Karte
        v-for="(karte, idx) in karteList"
        :key="`karte-${idx}`"
        :value="karte"
      />
    </section>
  </div>
</template>

<script>
import Karte from "./components/Karte.vue";

export default {
  name: "App",
  components: {
    Karte,
  },
  setup() {
    const karteList = [];

    for (let i = 0; i < 16; i++) {
      karteList.push(i);
    }

    return {
      karteList,
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

.karte {
  border: 3px solid rgb(116, 116, 116);
  border-radius: 10px;
}

.spielbrett {
  margin-top: 50px;
  display: grid;
  grid-template-columns: 90px 90px 90px 90px;
  grid-template-rows: 90px 90px 90px 90px;
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  justify-content: center;
}
</style>
