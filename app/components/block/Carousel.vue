<script lang="ts" setup>
import type { EmblaCarouselType } from 'embla-carousel'
import type { BlockCarousel } from '#storyblok-components'

interface Props {
  block: BlockCarousel
}

const { block } = defineProps<Props>()

const assets = computed(() => block.assets ?? [])

// Updated in onMounted from actual viewport width — avoids SSR/hydration mismatch
// while still generating enough slides for Embla's loop on any screen size.
const slideWidthPx = 28 // matches w-7 in the template
const minSlideCount = ref(200)

const slides = computed(() => {
  if (!assets.value.length) {
    return []
  }
  const repeats = Math.ceil(minSlideCount.value / assets.value.length)
  return Array.from({ length: repeats }).fill(assets.value).flat()
})

const autoScrollSpeed = computed(() => slideWidthPx / ((block.speed ?? 2) * 60))

const emblaRef = useTemplateRef('emblaRef')
const activeAsset = ref<(typeof assets.value)[number] | null>(null)
let embla: EmblaCarouselType | null = null

const updateActiveSlide = () => {
  if (!embla || !emblaRef.value || !assets.value.length) {
    return
  }

  const containerRect = emblaRef.value.getBoundingClientRect()
  const centerX = containerRect.left + containerRect.width / 2
  const slideNodes = embla.slideNodes()

  let closestIndex = 0
  let closestDistance = Infinity

  // Only check slides currently in the viewport — avoids iterating all 150–400 nodes
  for (const i of embla.slidesInView()) {
    const node = slideNodes[i]!
    const nodeCenterX = node.getBoundingClientRect().left + node.offsetWidth / 2
    const distance = Math.abs(nodeCenterX - centerX)

    if (distance < closestDistance) {
      closestDistance = distance
      closestIndex = i
    }
  }

  activeAsset.value = assets.value[closestIndex % assets.value.length] ?? null
}

async function initEmbla() {
  embla?.destroy()
  embla = null

  const [{ default: EmblaCarousel }, { default: AutoScroll }] = await Promise.all([
    import('embla-carousel'),
    import('embla-carousel-auto-scroll'),
  ])

  if (!emblaRef.value) { return }

  embla = EmblaCarousel(
    emblaRef.value,
    { loop: true, dragFree: true },
    [AutoScroll({ speed: autoScrollSpeed.value, stopOnInteraction: false, playOnInit: true })],
  )

  embla.on('scroll', updateActiveSlide)
}

onMounted(async () => {
  if (!emblaRef.value || !assets.value.length) { return }

  activeAsset.value = assets.value[0] ?? null

  // Ensure 3× the viewport width in slides so Embla's loop always has a valid
  // scroll range on any display size (1440p → ~155 slides, 4K → ~412 slides).
  minSlideCount.value = Math.ceil((window.innerWidth * 3) / slideWidthPx)

  // nextTick lets Vue commit the updated slide count to the DOM before Embla measures
  await nextTick()

  await initEmbla()
})

watch(autoScrollSpeed, initEmbla)

onUnmounted(() => {
  embla?.destroy()
})
</script>

<template>
  <div
    v-editable="block"
    class="relative w-full h-screen select-none"
  >
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
        playsinline
        autoplay
        muted
        loop
        class="absolute inset-0 size-full object-contain"
      />
    </template>

    <div
      ref="emblaRef"
      class="absolute inset-0 overflow-hidden"
    >
      <div class="flex h-full">
        <div
          v-for="(asset, index) in slides"
          :key="`${asset.id}-${index}`"
          class="w-7 h-full flex flex-col items-center justify-end"
        >
          <div
            v-if="asset?.filename"
            class="shrink-0 w-7 h-12 border border-white bg-white py-px"
          >
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
