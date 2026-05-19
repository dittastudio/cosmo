<script lang="ts" setup>
import type { EmblaCarouselType } from 'embla-carousel'
import type { BlockCarousel } from '#storyblok-components'

interface Props {
  block: BlockCarousel
}

const { block } = defineProps<Props>()

const assets = computed(() => block.assets ?? [])

// Repeat slides until we have at least 50 — ensures Embla's loop always has
// enough content to fill the viewport at any slide size.
const slides = computed(() => {
  if (!assets.value.length) return []
  const repeats = Math.ceil(50 / assets.value.length)
  return Array.from({ length: repeats }, () => assets.value).flat()
})

const emblaRef = ref<HTMLElement | null>(null)
const activeAsset = ref<(typeof assets.value)[number] | null>(null)
let embla: EmblaCarouselType | null = null

const updateActiveSlide = () => {
  if (!embla || !emblaRef.value || !assets.value.length) return
  const rect = emblaRef.value.getBoundingClientRect()
  const centerX = rect.left + rect.width / 2
  const slideNodes = embla.slideNodes()

  let closestIndex = 0
  let closestDistance = Infinity

  slideNodes.forEach((node, index) => {
    const nodeRect = node.getBoundingClientRect()
    const nodeCenterX = nodeRect.left + nodeRect.width / 2
    const distance = Math.abs(nodeCenterX - centerX)
    if (distance < closestDistance) {
      closestDistance = distance
      closestIndex = index
    }
  })

  activeAsset.value = assets.value[closestIndex % assets.value.length] ?? null
}

onMounted(async () => {
  if (!emblaRef.value || !assets.value.length) return

  activeAsset.value = assets.value[0] ?? null

  const [{ default: EmblaCarousel }, { default: AutoScroll }] = await Promise.all([
    import('embla-carousel'),
    import('embla-carousel-auto-scroll'),
  ])

  embla = EmblaCarousel(
    emblaRef.value,
    { loop: true, dragFree: true },
    [AutoScroll({ speed: 2, stopOnInteraction: false, playOnInit: true })],
  )

  embla.on('scroll', updateActiveSlide)
})

onUnmounted(() => {
  embla?.destroy()
})
</script>

<template>
  <div v-editable="block" class="relative w-full h-screen select-none">
    <template v-if="activeAsset?.filename">
      <NuxtImg
        v-if="storyblokAssetType(activeAsset.filename) === 'image'"
        :src="activeAsset.filename"
        width="800"
        class="absolute inset-0 size-full object-contain"
      />

      <video
        v-else-if="storyblokAssetType(activeAsset.filename) === 'video'"
        :key="activeAsset.filename"
        :src="activeAsset.filename"
        autoplay
        muted
        loop
        playsinline
        class="absolute inset-0 size-full object-contain"
      />
    </template>

    <div ref="emblaRef" class="absolute inset-0 overflow-hidden">
      <div class="flex h-full">
        <div
          v-for="(asset, index) in slides"
          :key="`${asset.id}-${index}`"
          class="w-7 h-full flex flex-col items-center justify-end"
        >
          <div v-if="asset?.filename" class="shrink-0 w-7 h-12 border border-white bg-white py-px">
            <NuxtImg
              v-if="storyblokAssetType(asset.filename) === 'image'"
              :src="asset.filename"
              width="60"
              class="size-full object-cover"
            />

            <video
              v-else-if="storyblokAssetType(asset.filename) === 'video'"
              :src="asset.filename"
              class="size-full object-cover"
            />
          </div>
        </div>
      </div>
    </div>

    <div class="absolute left-1/2 bottom-0.5 -translate-x-0.5 w-1 h-11 bg-red pointer-events-none" />
  </div>
</template>
