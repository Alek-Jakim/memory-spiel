<template>
  <div class="card">
    <div v-if="isShowing" class="card-side card-front">FRONT</div>
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
    isShowing: {
      type: Boolean,
      default: false,
    },
  },
};
</script>

<style>
.card {
  border: 3px solid rgb(116, 116, 116);
  border-radius: 10px;
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