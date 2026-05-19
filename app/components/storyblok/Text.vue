<script lang="ts" setup>
import type { BlockAttributes } from '@storyblok/richtext'
import type { Component, VNode } from 'vue'
import type { StoryblokRichtext } from '#storyblok-types'
import { Mark } from '@tiptap/core'
import { NuxtLink } from '#components'

interface Props {
  html?: StoryblokRichtext
}

const { html } = defineProps<Props>()

const CustomLink = Mark.create({
  name: 'link',
  renderHTML({ HTMLAttributes }) {
    const isEmail = HTMLAttributes.linktype === 'email'
    const isStory = HTMLAttributes.linktype === 'story'

    const attrs = {
      to: isEmail
        ? `mailto:${HTMLAttributes.href}`
        : isStory
          ? storyblokSlug(HTMLAttributes.href)
          : HTMLAttributes.href,
      target: HTMLAttributes.target || undefined,
    }

    return [asTag(NuxtLink), attrs, 0]
  },
})

type VNodeResult = VNode | VNode[]
type RenderChildren = VNodeResult | (() => VNodeResult)

const renderFn = (tag: string | Component, attrs: BlockAttributes, children?: RenderChildren): VNodeResult => {
  if (typeof tag !== 'string' && children != null) {
    const resolved = typeof children === 'function' ? children() : children
    return h(tag, attrs, { default: () => (Array.isArray(resolved) ? resolved : [resolved]) })
  }

  return h(tag as string, attrs, children as VNodeResult)
}

const { render } = useStoryblokRichText({
  renderFn,
  tiptapExtensions: {
    link: CustomLink,
  },
})

const richText = computed(() => (html ? () => render(html) : null))
</script>

<template>
  <component
    :is="richText"
    v-if="richText"
  />
</template>
