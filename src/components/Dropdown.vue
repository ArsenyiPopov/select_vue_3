<template>
  <ul class="dropdown">
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
  top: calc(100% + 10px); /* 10px spacing */
  left: 0;
  right: 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
  list-style: none;
  padding: 0;
  margin: 0;
  z-index: 1000;
  max-height: 250px;
  overflow-y: auto;

  .dropdown-item {
    padding: 8px;
    cursor: pointer;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;

    &:hover {
      background: #f0f0f0;
    }
  }
}
</style>

