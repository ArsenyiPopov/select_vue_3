<script setup>
import { ref } from 'vue';
import Select from './components/Select.vue';

const isError = ref(false);
const errorMessage = ref("Выберите компанию из списка");

// Функция валидации
const validateFunction = (value, items) => {
  if (!value) {
    return {
      isError: true,
      errorMessage: "Поле не должно быть пустым",
    };
  }

  const hasMatch = items.some(item => item.name.toLowerCase().includes(value.toLowerCase()));
  return {
    isError: !hasMatch,
    errorMessage: !hasMatch ? "Компания не найдена в списке" : "",
  };
};

// Пример данных
const items = [
  { name: 'Газпром', isError: true, errorMessage: 'Газпром недоступен для выбора', readOnly: false },
  { name: 'Норильский никель', isError: false, errorMessage: '', readOnly: false },
  { name: 'Татнефть', isError: false, errorMessage: '', readOnly: false },
  { name: 'Сахалинская энергия', isError: false},
  { name: 'Сбербанк', isError: false, errorMessage: '', readOnly: false },
  { name: 'Фосагро', isError: false, errorMessage: '', readOnly: false },
  { name: 'Арктикгаз', isError: false, errorMessage: '', readOnly: false },
  { name: 'Лаборатория Касперского', isError: false, errorMessage: '', readOnly: false },
  { name: 'Центральное КБ морской техники «Рубин»', isError: false, errorMessage: '', readOnly: false },
];
const itemsReadOnly = [
  { name: 'Газпром', isError: true, errorMessage: 'Газпром недоступен для выбора', readOnly: false },
  { name: 'Норильский никель', isError: false, errorMessage: '', readOnly: false },
  { name: 'Татнефть', isError: false, errorMessage: '', readOnly: false },
  { name: 'Сахалинская энергия', isError: true, errorMessage: 'Сахалинская все', readOnly: true },
  { name: 'Сбербанк', isError: false, errorMessage: '', readOnly: false },
  { name: 'Фосагро', isError: false, errorMessage: '', readOnly: false },
  { name: 'Арктикгаз', isError: false, errorMessage: '', readOnly: false },
  { name: 'Лаборатория Касперского', isError: false, errorMessage: '', readOnly: false },
  { name: 'Центральное КБ морской техники «Рубин»', isError: false, errorMessage: '', readOnly: false },
];
</script>

<template>
  <div>
    <!-- Первый компонент Select с данными isError: true -->
    <h1 class="select_title">Select Component Example Vue 3</h1>
    
    <Select
      :items="items"
      placeholder="Placeholder"
      title="Title:"
      hintIcon="ℹ️"
      hintText="Text example"
      :validateFn="validateFunction"
      :generalErrorMessage="errorMessage"
      :generalIsError="isError"
    />
<!-- Второй компонент Select с данными для ReadOnly -->
    
    <h2 class="readOnly_title">readOnly:true</h2>
    <Select
      :items="itemsReadOnly"
      placeholder="Placeholder"
      title="Title:"
      hintIcon="ℹ️"
      hintText="Text example"
      :validateFn="validateFunction"
      :generalErrorMessage="errorMessage"
      :generalIsError="isError"
    />
  </div>
</template>

<style scoped>
.select_title {
  margin-bottom: 30px;
}
.readOnly_title {
  margin-top: 50px;
  margin-bottom: 30px;
}

</style>
