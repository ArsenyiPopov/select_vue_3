<template>
  <div class="dropdown">
    <ul class="dropdown-content">
      <li
        v-for="item in items"
        :key="item.name"
        @mousedown.prevent="select(item)"
        :title="item.name"
        class="dropdown-item"
      >
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script setup>
const props = defineProps({
  items: Array
});

const emit = defineEmits(['select']);

const select = (item) => {
  emit('select', item);
};
</script>

<style lang="scss">
.dropdown {
  position: absolute;
  top: calc(100% + 6px); /* 10px spacing */
  left: 0;
  right: 0;
  border: 1px solid #ccc;
  border-radius: 14px;
  background-color: #EDEFF3;
  list-style: none;
  padding: 0;
  margin: 0;
  z-index: 1000;
  max-height: 250px;
  overflow: hidden; /* скрыть содержимое, выходящее за пределы */
}

.dropdown-content {
  max-height: inherit;
  overflow-y: auto;
  border-radius: inherit; /* для внутреннего контейнера */
  margin: 0;
  padding: 0;
}

.dropdown-item {
  padding: 8px;
  cursor: pointer;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;

  &:hover {
    background: #f5f7fb;
  }
}

::-webkit-scrollbar {
  width: 4px;
}

::-webkit-scrollbar-thumb {
  background-color: #bfc7d3;
}
</style>
