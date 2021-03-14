<template>
  <div class="select-bank__item" @click="$emit('active-bank')">
    <div class="select-bank__name"><slot></slot></div>
    <div class="select-bank__value">{{ rate }}%</div>
  </div>
</template>

<script>
export default {
  emits: ['active-bank'],
  data () {
    return {
      isActive: false
    }
  },
  props: {
    rate: {
      type: Number,
      required: true
    }
  }
}
</script>

<style scoped lang="scss">

.select-bank__item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 30px;
  max-width: 337px;
  border-radius: 5px;
  background-color: #ffffff;
  padding: 0 15px;
  margin-bottom: 15px;
  border: 1px solid #ccccd7;
  transition: 0.3s;

  &:hover {
    cursor: pointer;
    box-shadow: 2px 3px 10px rgba(0, 0, 0, .1);
  }

  &.active {
    background-color: #1adf2e1f;
    border: 1px solid #1adf2e;
  }
}
</style>
