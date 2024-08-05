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
          @input="onInput"
          :placeholder="currentPlaceholder"
          ref="selectInput"
          :title="searchQuery.length > maxInputLength ? searchQuery : ''"
          :readonly="isReadOnly"
        />
        <span v-if="hintIcon" class="hint-icon" @mouseover="showHint = true" @mouseleave="showHint = false">
          {{ hintIcon }}
          <span v-if="showHint" class="hint-text">{{ hintText }}</span>
        </span>
      </div>
      <span
        v-if="!isReadOnly"
        class="icon"
        :class="{ 'arrow-up': isOpen, 'arrow-down': !isOpen }"
        @click="toggleDropdown"
      ></span>
    </div>
    <p v-if="generalErrorState" class="error-message">{{ generalErrorMessage }}</p>
    <p v-if="companyErrorState" class="error-message">{{ companyErrorMessage }}</p>
    <Dropdown v-if="isOpen && !isReadOnly" :items="filteredItems" @select="selectItem" />
  </div>
</template>

<script setup>
import { ref, computed, nextTick, watch } from 'vue';
import Dropdown from './Dropdown.vue';

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
  placeholder: {
    type: String,
    default: 'Placeholder',
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
  validateFn: {
    type: Function,
    default: null,
  },
  generalErrorMessage: {
    type: String,
    default: 'Выберите компанию из списка',
  },
  generalIsError: {
    type: Boolean,
    default: false,
  },
});

const searchQuery = ref('');
const isOpen = ref(false);
const selectedValue = ref(null);
const showHint = ref(false);
const isItemSelected = ref(false);
const previousValue = ref('');
const currentPlaceholder = ref(props.placeholder);
const maxInputLength = 20;
const generalErrorState = ref(props.generalIsError);
const companyErrorState = ref(false);
const generalErrorMessage = ref(props.generalErrorMessage);
const companyErrorMessage = ref('');
const isReadOnly = computed(() => {
  // Check if there's a readOnly item to set the field as readonly
  const readOnlyItem = props.items.find(item => item.readOnly);
  if (readOnlyItem) {
    searchQuery.value = readOnlyItem.name;
    return true;
  }
  return false;
});

const filterItems = () => {
  if (!isOpen.value && !isReadOnly.value) {
    isOpen.value = true;
  }
};

const onInput = () => {
  if (isReadOnly.value) return; // Prevent further input when readOnly
  filterItems();
  validateInput(searchQuery.value);
};

const onFocus = () => {
  if (isReadOnly.value) return; // Prevent focus when readOnly
  previousValue.value = searchQuery.value;
  if (isItemSelected.value) {
    searchQuery.value = '';
    isItemSelected.value = false;
  }
  isOpen.value = true;
  currentPlaceholder.value = props.searchPlaceholder;
};

const onBlur = () => {
  nextTick(() => {
    if (!isItemSelected.value && !isReadOnly.value) {
      searchQuery.value = previousValue.value;
      isOpen.value = false;
      currentPlaceholder.value = props.placeholder;
    }
  });
};

const selectItem = (item) => {
  if (item.readOnly) return; // Prevent selection when readOnly

  selectedValue.value = item.name;
  searchQuery.value = item.name;
  isItemSelected.value = true;
  isOpen.value = false;
  currentPlaceholder.value = props.placeholder;

  if (item.isError) {
    companyErrorState.value = true;
    companyErrorMessage.value = item.errorMessage;
  } else {
    companyErrorState.value = false;
    companyErrorMessage.value = '';
  }
  generalErrorState.value = false;
};

const toggleDropdown = () => {
  if (isReadOnly.value) return; // Prevent toggling when readOnly
  isOpen.value = !isOpen.value;
};

const filteredItems = computed(() => {
  return props.items
    .filter(item => item.name.toLowerCase().includes(searchQuery.value.toLowerCase()))
    .filter(item => !item.readOnly); // Exclude readOnly items from dropdown
});

const validateInput = (input) => {
  if (props.validateFn) {
    const validationResult = props.validateFn(input, props.items);
    generalErrorState.value = validationResult.isError;
    generalErrorMessage.value = validationResult.errorMessage;
  } else {
    if (!input) {
      generalErrorState.value = true;
      generalErrorMessage.value = 'Выберите компанию из списка';
    } else {
      const hasMatch = props.items.some(item => item.name.toLowerCase().includes(input.toLowerCase()));
      generalErrorState.value = !hasMatch;
      generalErrorMessage.value = !hasMatch ? props.generalErrorMessage : '';
    }
  }

  const matchingCompany = props.items.find(item => item.name.toLowerCase() === input.toLowerCase());
  if (matchingCompany && matchingCompany.isError) {
    companyErrorState.value = true;
    companyErrorMessage.value = matchingCompany.errorMessage;
  } else {
    companyErrorState.value = false;
    companyErrorMessage.value = '';
  }
};

watch(() => props.generalIsError, (newVal) => {
  generalErrorState.value = newVal;
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
  background-color: var(--input-bg, white);
  cursor: var(--input-cursor, text);

  &::placeholder {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  &:read-only {
    background-color: #f0f0f0; /* Gray background for read-only */
    cursor: not-allowed; /* Not-allowed cursor for read-only */
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
  bottom: -20px;
  right: -60px;
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

.error-message {
  color: red;
  font-size: 0.75rem;
  margin-top: 4px;
}
</style>
