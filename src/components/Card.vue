<template>
  <div class="card" @click="toggleSelectCard">
    <div v-if="isVisible" class="card-side card-front">
      {{ value }} - {{ position }}
    </div>
    <div v-else class="card-side card-back">BACK</div>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: Number,
      required: true,
    },
    position: {
      type: Number,
      required: true,
    },
    isVisible: {
      type: Boolean,
      default: false,
    },
    isCorrect: {
      type: Boolean,
      default: false,
    },
  },
  setup(props, context) {
    //Here I want to emit an event to the parent element, to know which card has been selected so we can pass the "props, context" arguments to the setup method
    const toggleSelectCard = () => {
      context.emit("select-card", {
        position: props.position,
        positionValue: props.value,
      });
    };

    return {
      toggleSelectCard,
    };
  },
};
</script>

<style>
.card {
  border: 3px solid rgb(116, 116, 116);
  position: relative;
}

.card-side {
  width: 100%;
  height: 100%;
  position: absolute;
}

.card-side.card-front {
  background: red;
  color: #fff;
}

.card-side.card-back {
  background: blue;
  color: #fff;
}
</style>