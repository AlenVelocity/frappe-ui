<template>
  <div class="block w-full">
    <div
      class="flex items-start rounded-md px-4 py-3.5 text-base md:px-5"
      :class="classes"
    >
      <svg
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          opacity="0.8"
          fill-rule="evenodd"
          clip-rule="evenodd"
          d="M12 2C6.5 2 2 6.5 2 12C2 17.5 6.5 22 12 22C17.5 22 22 17.5 22 12C22 6.5 17.5 2 12 2ZM12 10.5C12.5523 10.5 13 10.9477 13 11.5V17C13 17.5523 12.5523 18 12 18C11.4477 18 11 17.5523 11 17V11.5C11 10.9477 11.4477 10.5 12 10.5ZM13 7.99976C13 7.44747 12.5523 6.99976 12 6.99976C11.4477 6.99976 11 7.44747 11 7.99976V8.1C11 8.65228 11.4477 9.1 12 9.1C12.5523 9.1 13 8.65228 13 8.1V7.99976Z"
          fill="#318AD8"
        />
      </svg>
      <div class="ml-2 w-full">
        <div class="flex flex-col md:flex-row md:items-baseline">
          <h3 class="text-lg font-medium text-ink-gray-9" v-if="title">
            {{ title }}
          </h3>
          <div class="mt-1 md:ml-2 md:mt-0">
            <slot></slot>
          </div>
          <div class="mt-3 md:ml-auto md:mt-0">
            <slot name="actions"></slot>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { AlertProps } from './types'

const props = withDefaults(defineProps<AlertProps>(), {
  type: 'warning',
})

const classes = computed(() => {
  return {
    warning: 'text-ink-gray-7 bg-surface-blue-1',
  }[props.type]
})
</script>
