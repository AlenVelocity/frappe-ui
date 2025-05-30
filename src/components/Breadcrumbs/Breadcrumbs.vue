<template>
  <div class="flex min-w-0 items-center">
    <template v-if="dropdownItems.length">
      <Dropdown class="h-7" :options="dropdownItems">
        <Button variant="ghost">
          <template #icon>
            <svg
              class="w-4 text-ink-gray-5"
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <circle cx="12" cy="12" r="1" />
              <circle cx="19" cy="12" r="1" />
              <circle cx="5" cy="12" r="1" />
            </svg>
          </template>
        </Button>
      </Dropdown>
      <span class="ml-1 mr-0.5 text-base text-ink-gray-4" aria-hidden="true">
        /
      </span>
    </template>
    <div
      class="flex min-w-0 items-center overflow-hidden text-ellipsis whitespace-nowrap"
    >
      <template v-for="(item, i) in crumbs" :key="item.label">
        <router-link
          v-if="item.route"
          :to="item.route"
          @click="item.onClick ? item.onClick() : null"
          class="flex items-center rounded px-0.5 py-1 text-lg font-medium focus:outline-none focus-visible:ring-2 focus-visible:ring-outline-gray-3"
          :class="[
            i == crumbs.length - 1
              ? 'text-ink-gray-9'
              : 'text-ink-gray-5 hover:text-ink-gray-7',
          ]"
        >
          <slot name="prefix" :item="item" />
          <span>
            {{ item.label }}
          </span>
          <slot name="suffix" :item="item" />
        </router-link>
        <button
          v-else
          @click="item.onClick ? item.onClick() : null"
          class="flex items-center rounded px-0.5 py-1 text-lg font-medium focus:outline-none focus-visible:ring-2 focus-visible:ring-outline-gray-3"
          :class="[
            i == crumbs.length - 1
              ? 'text-ink-gray-9'
              : 'text-ink-gray-5 hover:text-ink-gray-7',
          ]"
        >
          <slot name="prefix" :item="item" />
          <span>
            {{ item.label }}
          </span>
          <slot name="suffix" :item="item" />
        </button>
        <span
          v-if="i != crumbs.length - 1"
          class="mx-0.5 text-base text-ink-gray-4"
          aria-hidden="true"
        >
          /
        </span>
      </template>
    </div>
  </div>
</template>
<script setup lang="ts">
import { useWindowSize } from '@vueuse/core'
import { computed } from 'vue'
import { useRouter } from 'vue-router'
import { Dropdown } from '../Dropdown'
import { Button } from '../Button'
import type { BreadcrumbsProps } from './types'

const props = defineProps<BreadcrumbsProps>()

const router = useRouter()
const { width } = useWindowSize()

const items = computed(() => {
  return (props.items || []).filter(Boolean)
})

const dropdownItems = computed(() => {
  if (width.value > 640) return []

  let allExceptLastTwo = items.value.slice(0, -2)
  return allExceptLastTwo.map((item) => {
    let onClick = () => {
      if (item.onClick) {
        item.onClick()
      }
      if (item.route) {
        router.push(item.route)
      }
    }
    return {
      ...item,
      icon: null,
      label: item.label,
      onClick,
    }
  })
})

const crumbs = computed(() => {
  if (width.value > 640) return items.value

  let lastTwo = items.value.slice(-2)
  return lastTwo
})
</script>
