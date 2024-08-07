<template>
  <div class="select-container">
    <div class="select-input-wrapper">
      <label 
        v-if="title"
        class="select-title"
        :class="{ 'error-color': generalErrorState || companyErrorState }">
        {{ title }}</label>
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
          :class="{ 'error-border': generalErrorState || companyErrorState, 'error-text': generalErrorState || companyErrorState }"
        />
        <span v-if="hintIcon" class="hint-icon" @mouseover="showHint = true" @mouseleave="showHint = false">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
  <path fill-rule="evenodd" clip-rule="evenodd" d="M9 16.3632C13.2924 16.3632 16.3632 13.2924 16.3632 9C16.3632 4.7076 13.2924 1.6368 9 1.6368C4.7076 1.6368 1.6368 4.7076 1.6368 9C1.6368 13.2924 4.7076 16.3632 9 16.3632ZM9 18C6.61305 18 4.32387 17.0518 2.63604 15.364C0.948212 13.6761 0 11.3869 0 9C0 6.61305 0.948212 4.32387 2.63604 2.63604C4.32387 0.948212 6.61305 0 9 0C11.3869 0 13.6761 0.948212 15.364 2.63604C17.0518 4.32387 18 6.61305 18 9C18 11.3869 17.0518 13.6761 15.364 15.364C13.6761 17.0518 11.3869 18 9 18Z" fill="#828D9B"/>
  <path d="M8.08967 14V7.45455H9.90501V14H8.08967ZM9.0016 6.1108C8.73171 6.1108 8.50018 6.02131 8.307 5.84233C8.11665 5.66051 8.02148 5.44318 8.02148 5.19034C8.02148 4.94034 8.11665 4.72585 8.307 4.54688C8.50018 4.36506 8.73171 4.27415 9.0016 4.27415C9.27148 4.27415 9.5016 4.36506 9.69194 4.54688C9.88512 4.72585 9.98171 4.94034 9.98171 5.19034C9.98171 5.44318 9.88512 5.66051 9.69194 5.84233C9.5016 6.02131 9.27148 6.1108 9.0016 6.1108Z" fill="#828D9B"/>
</svg>
        </span>
        <span v-if="showHint" class="hint-text">{{ hintText }}</span>
      </div>
      <span
        v-if="!isReadOnly"
        class="icon"
        :class="{ 'arrow-up': isOpen, 'arrow-down': !isOpen }"
        @click="toggleDropdown"
      >
      <svg v-if="!isOpen" width="13" height="10" viewBox="0 0 13 10" fill="none" xmlns="http://www.w3.org/2000/svg">
  <path d="M12 3L6.5 8L1 3" stroke="#292929" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</svg>
<svg v-else width="13" height="10" viewBox="0 0 13 10" fill="none" xmlns="http://www.w3.org/2000/svg">
  <path d="M1 7L6.5 2L12 7" stroke="#292929" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</svg>
      </span>
    </div>
    <p v-if="generalErrorState" class="error-message">{{ generalErrorMessage }}</p>
    <p v-if="companyErrorState" class="error-message">{{ companyErrorMessage }}</p>
    <Dropdown v-if="isOpen && !isReadOnly && filteredItems.length" :items="filteredItems" @select="selectItem" />
  </div>
</template>


<script setup>
import { ref, computed, nextTick, watch } from 'vue';
import Dropdown from './Dropdown.vue';

const props = defineProps({
  items: Array,
  placeholder: String,
  searchPlaceholder: String,
  title: String,
  hintIcon: String,
  hintText: String,
  validateFn: Function,
  generalErrorMessage: String,
  generalIsError: Boolean,
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
  if (isReadOnly.value) return;
  filterItems();
  validateInput(searchQuery.value);
};

const onFocus = () => {
  if (isReadOnly.value) return;
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
      // Validate the old value to clear any lingering error states
      validateInput(previousValue.value);
    }
  });
};

const selectItem = (item) => {
  if (item.readOnly) return;

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

  nextTick(() => {
    const input = document.querySelector('.select-input');
    if (input) input.blur();
  });
};

const toggleDropdown = () => {
  if (isReadOnly.value) return;
  isOpen.value = !isOpen.value;
};

const filteredItems = computed(() => {
  return props.items
    .filter(item => item.name.toLowerCase().includes(searchQuery.value.toLowerCase()))
    .filter(item => !item.readOnly);
});


const validateInput = (input) => {
  if (props.validateFn) {
    const { isError, errorMessage } = props.validateFn(input, props.items);
    generalErrorState.value = isError;
    generalErrorMessage.value = errorMessage || '';
  } else {
    if (!input) {
      generalErrorState.value = true;
      generalErrorMessage.value = 'Поле не должно быть пустым';
    } else {
      const hasMatch = props.items.some(item => item.name.toLowerCase() === input.toLowerCase());
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
  width: 320px;
}

.select-input-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
}

.select-title {
  position: absolute;
  left: 12px;
  font-size: 13px;
  pointer-events: none;
  font-weight: bold;
  z-index: 1;
  color: #292929;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.select-input {
  width: 100%;
  padding: 20px 46px 8px 12px;
  border: 1px solid #EDEFF3;
  border-radius: 14px;
  box-sizing: border-box;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  background-color: #EDEFF3;
  cursor: var(--input-cursor, text);

  &::placeholder {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    font-family: 'Source Sans Pro', sans-serif;
    color: #A6AEB8;
    font-size: 16px;
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
  right: 32px;
  cursor: pointer;
  top: 14px;
}

.hint-text {
  position: absolute;
  bottom: -20px;
  right: 0px;
  background: #daebfe;
  color: #2b68bc;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 0.75rem;
  white-space: nowrap;
  z-index: 1001;
}

.icon {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
}

.error-message {
  color: #D45050;
  font-size: 12px;
  margin-left: 12px;
}
.select-title.error-color {
  color: #D45050;
}
.select-input.error-text {
  color: #D45050;
}
</style>
