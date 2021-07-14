<template>
  <div class="card" :class="flipCard" @click="toggleSelectCard">
    <div class="card-side card-front">
      <img :src="`/pictures/${value}.png`" :alt="value" />
    </div>
    <div class="card-side card-back"></div>
  </div>
</template>

<script>
import { computed } from "vue";
export default {
  props: {
    value: {
      type: String,
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
    isStarted: {
      type: Boolean,
      default: false,
    },
  },
  setup(props, context) {
    const flipCard = computed(() => {
      if (props.isVisible) return "flip-card";
      return "";
    });

    //Here I want to emit an event to the parent element, to know which card has been selected so we can pass the "props, context" arguments to the setup method
    const toggleSelectCard = () => {
      context.emit("select-card", {
        position: props.position,
        positionValue: props.value,
      });
    };

    return {
      flipCard,
      toggleSelectCard,
    };
  },
};
</script>

<style>
.card {
  position: relative;
  transition: 0.2s transform ease-in-out;
  transform-style: preserve-3d;
}

.card.flip-card {
  transform: rotateY(180deg);
}

.card-side.card-front {
  transform: rotateY(180deg);
  width: 100%;
  height: 100%;
  background-color: #000;
}

.card-side {
  width: 100%;
  height: 100%;
  position: absolute;
  border: 2px solid #000;
}

.card-side.card-back {
  background-image: url("/pictures/q-mark.png");
  background-color: rgb(185, 185, 185);
  width: 100%;
  height: 100%;
  background-size: contain;
}
</style>