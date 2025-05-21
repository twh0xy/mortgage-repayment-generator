<script setup lang="ts">
import { ref } from 'vue'
import ContainerLeft from './components/sideleft/ContainerLeft.vue'
import ContainerRight from './components/sideright/ContainerRight.vue'

const result = ref<{ monthly: number; total: number } | null>(null)
</script>

<template>
  <div class="container-app">
    <div class="mortgage_cl_left">
      <ContainerLeft @calculated="result = $event" @clear="result = null" />
    </div>
    <div class="mortgage_cl_right">
      <ContainerRight :result="result" />
    </div>
  </div>
</template>

<style scoped>
.container-app {
  display: flex;
  align-items: center;
  width: 1000px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 24px;
  border-radius: 0px 20px 20px 0px;

  & > .mortgage_cl_left {
    width: 460px;
    padding: 14px;
    background-color: hsl(0, 0%, 100%);
    border: 1px solid hsla(0, 0%, 14%, 0.164);
    border-radius: 20px 0px 0px 20px;
  }

  & > .mortgage_cl_right {
    width: 560px;
    height: 474px;
    padding: 14px;
    background-color: hsl(201, 54%, 16%);
    border: 1px solid hsla(0, 0%, 14%, 0.164);
    border-radius: 0px 20px 20px 0px;
  }
}

@media (max-width: 1010px) {
  .container-app {
    display: flex;
    align-items: center;
    flex-direction: column;
    width: 100%;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 24px;
    border-radius: 0px 20px 20px 0px;

    & > .mortgage_cl_right {
      width: 460px;
      height: 474px;
      padding: 14px;
      background-color: hsl(201, 54%, 16%);
      border: 1px solid hsla(0, 0%, 14%, 0.164);
      border-radius: 0px 20px 20px 0px;
    }
  }
}
</style>
