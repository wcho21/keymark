<script lang="ts" setup>
import { marked } from "marked";
import type { HooksObject } from "marked";
import { debounce, range } from "lodash-es";
import { ref, computed } from "vue";

const slideBeg = "<div class=\"markdown-slide\">";
const slideEnd = "</div>";

const slidify = (regex: RegExp, source: string) => {
  const begs = [0, ...Array.from(source.matchAll(regex)).map(match => match.index), source.length-1];
  const sliced = range(begs.length-1).map(i => source.slice(begs[i], begs[i+1]));
  return slideBeg + sliced.join(slideEnd + slideBeg) + slideEnd;
};

const hooks: HooksObject = {
  postprocess(markdown) {
    const h2Slides = slidify(/(\<h1\>|\<h2\>)/g, markdown);
    console.log(h2Slides);

    return h2Slides;
  },
}
marked.use({ hooks });

const markdownSource = ref<string>("");
const parsed = computed(() => marked(markdownSource.value));
const updateSource = debounce((e) => {
  markdownSource.value = e.target.value;
}, 50);
</script>

<template>
  <div class="w-full h-screen bg-slate-100 flex flex-col">
    <div class="w-full h-12 flex-0 flex gap-2 items-center bg-blue-100">
      <h1 class="px-2"><a href="#">Keymark (heading)</a></h1>
      <button class="ml-auto h-full px-2">Fullscreen</button>
      <button class="px-2 h-full">Share</button>
    </div>
    <div class="flex-1 bg-blue-200 flex h-full">
      <div class="flex-1 h-full editor-component w-50">
        <textarea :value="markdownSource" @input="updateSource" class="p-2 w-full h-full resize-none"></textarea>
      </div>
      <div class="flex-1 h-full preview-component p-2" v-html="parsed"></div>
    </div>
  </div>
</template>

<style lang="postcss" scoped>
.preview-component :deep(h1) {
  @apply font-bold text-3xl;
}

.preview-component :deep(h2) {
  @apply font-bold text-2xl;
}

.preview-component :deep(h3) {
  @apply font-bold text-xl;
}

.preview-component :deep(h4) {
  @apply font-bold text-lg;
}

.preview-component :deep(h5) {
  @apply font-bold;
}

.preview-component :deep(.markdown-slide) {
  @apply border;
}
</style>
