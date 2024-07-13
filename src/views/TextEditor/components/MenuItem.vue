<template>
  <button
    class="menu-item"
    :class="{ 'is-active': item.isActive ? item.isActive() : null }"
    @click="item.action"
    :title="item.title"
    >
    <svg class="remix">
      <use :xlink:href="`${remixiconUrl}#ri-${item.icon}`" />
    </svg>
  </button>
</template>

<script setup lang="ts" name="MenuItem">
  import remixiconUrl from 'remixicon/fonts/remixicon.symbol.svg'
  import { MenuItemType } from "../../../types"
  
  const {item} = defineProps<{ item: MenuItemType,}>()
</script>

<style lang="scss" scoped>
  .menu-item {
    background: transparent;
    border: none;
    border-radius: 0.4rem;
    color: #333;
    cursor: pointer;
    height: 1.75rem;
    padding: 0.25rem;
    margin-right: 0.25rem;
    width: 1.75rem;

    svg {
      fill: currentColor;
      height: 100%;
      width: 100%;
    }

    &.is-active,
    &:hover {
      background-color: #d6d6d6;
    }
  }
</style>
