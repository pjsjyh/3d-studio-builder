<template>
  <div class="h-100 align-items-center d-flex" style="flex: 1; overflow: hidden" :style="{ display: isVisible ? 'none !important' : 'block' }">
    <div class="w-100 d-flex align-items-center" style="height: 40%; flex-direction: column; flex: 1">
      <v-icon
        class="builder-side-box-search-icon-big"
        :class="getImg"
        :style="{ fontSize: imgSize, display: getImg === '' ? 'none' : 'flex' }"
        style="width: 8rem; height: 6rem"
        color="#7E7E7E"
        x-large
        >{{ getImg.includes('mdi') ? getImg : '' }}
      </v-icon>
      <p style="font-size: 0.8125rem; color: #7e7e7e; text-align: center" v-html="convertLineBreaks(getWord)"></p>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    getWord: { type: String, default: '' },
    getImg: { type: String, default: '' },
    imgSize: { type: String, default: '68px' },
    isVisible: { type: Boolean, default: false },
  },
  methods: {
    convertLineBreaks(inputText) {
      return inputText.replace(/\n/g, '<br>')
    },
  },
}
</script>

<style></style>
