<script lang="ts" setup>
import type { ISbStoryData } from '@storyblok/js'
import type { Page } from '#storyblok-components'

interface Props {
  story: ISbStoryData<Page>
}

const { story } = defineProps<Props>()
</script>

<template>
  <section
    v-for="block in story.content.blocks"
    :key="block._uid"
  >
    <BlockCarousel
      v-if="block.component === 'block_carousel'"
      :block="block"
    />
  </section>
</template>

<style scoped>
@reference "@/assets/css/app.css";
</style>
