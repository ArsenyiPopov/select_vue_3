<script setup>
import Select from './components/Select.vue';
</script>

<template>
  <div>
    <h1>Select Component Example Vue 3</h1>
    <Select
      :items="['Газпром', 'Норильский никель', 'Татнефть', 'Сахалинская энергия', 'Сбербанк', 'Фосагро', 'Арктикгаз','Лаборатория Касперского','Центральное КБ морской техники «Рубин»']"
      placeholder="Placeholder"
      title="title:"
      hintIcon="ℹ️"
      hintText="Choose your company"
    />
  </div>
</template>

<style scoped>





</style>
