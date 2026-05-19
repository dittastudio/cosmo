<script lang="ts" setup>
import type { BlockCarousel } from '#storyblok-components'

interface Props {
  block: BlockCarousel
}

const { block } = defineProps<Props>()
</script>

<template>
  <div v-editable="block" class="w-full h-screen bg-[red]">
    <ul>
      <li v-for="asset in block.assets">
        <template v-if="asset?.filename">
          <div v-if="storyblokAssetType(asset.filename) === 'image'">
            <NuxtImg :src="asset.filename" />
          </div>

          <div v-else-if="storyblokAssetType(asset.filename) === 'video'">
            <video :src="asset.filename" />
          </div>
        </template>
      </li>
    </ul>
  </div>
</template>
