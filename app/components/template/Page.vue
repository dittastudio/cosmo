<script lang="ts" setup>
import type { ISbStoryData } from '@storyblok/js'
import type { Page } from '#storyblok-components'

interface Props {
  story: ISbStoryData<Page>
}

const { story } = defineProps<Props>()
</script>

<template>
  <div v-if="story.content.text" class="absolute top-0 left-0 z-10 p-5">
    <StoryblokText :html="story.content.text" />
  </div>

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
