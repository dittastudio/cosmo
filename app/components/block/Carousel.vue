<script lang="ts" setup>
import 'keen-slider/keen-slider.min.css'
import { useKeenSlider } from 'keen-slider/vue'
import type { BlockCarousel } from '#storyblok-components'

interface Props {
  block: BlockCarousel
  perView?: number | 'auto'
}

const { block, perView = 'auto' } = defineProps<Props>()

const [container] = useKeenSlider({
  loop: true,
  drag: true,
  slides: {
    perView: () => perView,
    spacing: 0,
  },
})
</script>

<template>
  <div v-editable="block" class="w-full h-screen flex flex-col">
    <div class="w-full bg-[green] grow">
      MEDIA
    </div>

    <div ref="container" class="keen-slider h-12">
      <div
        v-for="(asset, index) in block.assets"
        :key="`${asset.id}-${index}`"
        class="keen-slider__slide size-12!"
      >
        <template v-if="asset?.filename">
          <NuxtImg
            v-if="storyblokAssetType(asset.filename) === 'image'"
            :src="asset.filename"
            class="size-full object-cover"
          />
          <video
            v-else-if="storyblokAssetType(asset.filename) === 'video'"
            :src="asset.filename"
            autoplay
            muted
            loop
            playsinline
            class="size-full object-cover"
          />
        </template>
      </div>
    </div>
  </div>
</template>
