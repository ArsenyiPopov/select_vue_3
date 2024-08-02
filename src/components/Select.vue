<template>
    <div class="select-container">
      <div class="select-input-wrapper">
        <label v-if="title" class="select-title">{{ title }}</label>
        <div class="input-wrapper">
          <input
            type="text"
            class="select-input"
            v-model="searchQuery"
            @focus="onFocus"
            @blur="onBlur"
            @input="filterItems"
            :placeholder="currentPlaceholder"
            ref="selectInput"
            :title="searchQuery.length > maxInputLength ? searchQuery : ''"
          />
          <span v-if="hintIcon" class="hint-icon" @mouseover="showHint = true" @mouseleave="showHint = false">
            {{ hintIcon }}
            <span v-if="showHint" class="hint-text">{{ hintText }}</span>
          </span>
        </div>
      </div>
      <span class="icon" :class="{ 'arrow-up': isOpen, 'arrow-down': !isOpen }" @click="toggleDropdown"></span>
      <Dropdown v-if="isOpen" :items="filteredItems" @select="selectItem" />
    </div>
  </template>
  
  <script setup>
  import { ref, computed, nextTick } from 'vue';
  import Dropdown from './Dropdown.vue';
  
  const props = defineProps({
    items: {
      type: Array,
      required: true,
    },
    placeholder: {
      type: String,
      default: 'Select a fruit',
    },
    searchPlaceholder: {
      type: String,
      default: 'Search',
    },
    title: {
      type: String,
      default: '',
    },
    hintIcon: {
      type: String,
      default: '',
    },
    hintText: {
      type: String,
      default: '',
    },
  });
  
  const searchQuery = ref('');
  const isOpen = ref(false);
  const selectedValue = ref(null);
  const showHint = ref(false);
  const isItemSelected = ref(false);
  const previousValue = ref('');
  const currentPlaceholder = ref(props.placeholder); // Динамический плейсхолдер
  const maxInputLength = 20; // Максимальная длина значения в поле ввода
  
  const filterItems = () => {
    if (!isOpen.value) {
      isOpen.value = true;
    }
  };
  
  const onFocus = () => {
    previousValue.value = searchQuery.value;
    if (isItemSelected.value) {
      searchQuery.value = '';
      isItemSelected.value = false;
    }
    isOpen.value = true;
    currentPlaceholder.value = props.searchPlaceholder; // Изменяем плейсхолдер на "Search"
  };
  
  const onBlur = () => {
    nextTick(() => {
      if (!isItemSelected.value) {
        searchQuery.value = previousValue.value;
        isOpen.value = false;
        currentPlaceholder.value = props.placeholder; // Возвращаем первоначальный плейсхолдер
      }
    });
  };
  
  const selectItem = (item) => {
    selectedValue.value = item;
    searchQuery.value = item;
    isItemSelected.value = true;
    isOpen.value = false;
    currentPlaceholder.value = props.placeholder; // Возвращаем первоначальный плейсхолдер после выбора элемента
  };
  
  const toggleDropdown = () => {
    isOpen.value = !isOpen.value;
  };
  
  const filteredItems = computed(() => {
    return props.items.filter(item => item.toLowerCase().includes(searchQuery.value.toLowerCase()));
  });
  </script>
  
  <style lang="scss">
  .select-container {
    position: relative;
    width: 300px;
  }
  
  .select-input-wrapper {
    position: relative;
    display: flex;
    flex-direction: column;
  }
  
  .select-title {
    position: absolute;
    left: 4px;
    font-size: 0.75rem;
    color: #666;
    pointer-events: none;
    transition: 0.2s;
    font-weight: bold;
    z-index: 1;
  }
  
  .input-wrapper {
    position: relative;
    display: flex;
    align-items: center;
  }
  
  .select-input {
    width: 100%;
    padding: 16px 46px 16px 4px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  
    &::placeholder {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  }
  
  .select-input:focus + .select-title,
  .select-input:not(:placeholder-shown) + .select-title {
    top: -10px;
    font-size: 0.65rem;
    color: #007bff;
  }
  
  .hint-icon {
    position: absolute;
    right: 26px;
    top: 50%;
    transform: translateY(-50%);
    cursor: pointer;
  }
  
  .hint-text {
    position: absolute;
    top: -20px;
    right: 45px;
    background: #000;
    color: #fff;
    padding: 2px 8px;
    border-radius: 4px;
    font-size: 0.75rem;
    white-space: nowrap;
    z-index: 1;
  }
  
  .icon {
    position: absolute;
    right: 10px;
    top: 50%;
    transform: translateY(-50%);
    cursor: pointer;
  }
  
  .icon.arrow-up::before {
    content: '▲';
  }
  
  .icon.arrow-down::before {
    content: '▼';
  }
  </style>
  