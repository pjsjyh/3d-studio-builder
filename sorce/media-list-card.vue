<template>
  <section class="f-column w-100 f-full">
    <div v-show="isGrid">
      <div
        class="modelview-grid align-items-start"
        style="display: grid; grid-template-columns: repeat(2, 1fr); overflow-y: auto; overflow-x: hidden; height: fit-content"
      >
        <v-card
          v-for="(item, index) in downloadModelingList"
          :key="item.properties.filename"
          class="modelview-card d-flex flex-column align-items-center justify-content-center border-0"
          style="width: 8.35714rem; height: 9.14286rem; cursor: pointer"
          outlined
          :style="{
            display: item.onoff ? (checkModalTagIn(item, selectedTagList) ? 'flex' : 'none !important') : 'none !important',
            backgroundColor: darkMode ? '#2A2A2A !important' : 'rgba(240, 241, 242, 0.5) !important',
          }"
          @click="clickComponent(item)"
        >
          <div class="modelview-card-viewer mt-3 rounded bg-white" style="width: 6.92857rem; height: 4.92857rem; border-radius: 0.35714rem">
            <template v-if="checkType(item.properties.mimetype) === 'model'">
              <template v-if="item.properties.thumbnailPath">
                <model-viewer
                  :ref="`modelViewer-${index}-card`"
                  reveal="manual"
                  :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.path}`"
                  style="display: block; width: 100%; height: 100%"
                  camera-controls
                >
                  <v-img
                    id="lazy-load-poster"
                    ref="modelViewerPoster"
                    slot="poster"
                    class="position-absolute"
                    :style="{ backgroundImage: 'url(' + `${gractorBaseUrl}/api/media/download?path=${item.properties.thumbnailPath}` + ')' }"
                    @mouseover="
                      () => {
                        captureAndSetPoster(index, 'card')
                      }
                    "
                  ></v-img
                  ><v-icon color="#FFD600" style="position: absolute; right: 0" :style="{ display: item.fav ? 'block' : 'none' }"
                    >mdi-star</v-icon
                  ></model-viewer
                >
              </template>
              <template v-else>
                <model-viewer
                  :ref="`modelViewer-${index}-card`"
                  :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.path}`"
                  style="display: block; width: 100%; height: 100%"
                  camera-controls
                  ><v-icon color="#FFD600" style="position: absolute; right: 0" :style="{ display: item.fav ? 'block' : 'none' }"
                    >mdi-star</v-icon
                  ></model-viewer
                >
              </template>
            </template>
            <template v-if="checkType(item.properties.mimetype) === 'photo'">
              <v-img
                class="modelview-card-viewer-img"
                :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.thumbnailPath}`"
                style="width: 100%; height: 100%; border-radius: 0.1875rem"
              />
            </template>
            <template v-if="checkType(item.properties.mimetype) === 'video'">
              <iframe
                width="100%"
                height="100%"
                :src="item.properties.srcPath + '?autoplay=1'"
                frameborder="0"
                allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
                allowfullscreen
              ></iframe>
              <!-- <v-img class="modelview-card-viewer-img" :src="item.thumbnailname" style="width: 100%; height: 100%; border-radius: 0.1875rem" /> -->
            </template>
          </div>
          <p class="modelview-card-viewer-text modelview-card-title py-3 d-flex" style="font-size: 0.8125rem">{{ truncateFilename(item.properties.name) }}</p>
        </v-card>
      </div>
    </div>
    <div v-show="!isGrid">
      <div class="modelview-list d-flex flex-column align-items-start" style="height: fit-content; overflow-y: auto; overflow-x: hidden">
        <v-list class="w-100">
          <v-list-item
            v-for="(item, index) in downloadModelingList"
            :key="item.properties.filename"
            class="modelview-layer w-100 d-flex align-items-center position-relative g-cursor-pointer"
            style="border-bottom: 1px solid #f0f0f0; padding: 0.6rem 1rem; border-radius: 0.35714rem"
            :style="{ display: item.onoff ? (checkModalTagIn(item, selectedTagList) ? 'flex' : 'none !important') : 'none !important' }"
            @click="clickComponent(item)"
          >
            <div style="width: 4.5rem; height: 3rem">
              <template v-if="checkType(item.properties.mimetype) === 'model'">
                <template v-if="item.properties.thumbnailPath">
                  <model-viewer
                    :ref="`modelViewer-${index}-card`"
                    reveal="manual"
                    :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.path}`"
                    style="display: block; width: 100%; height: 100%"
                    camera-controls
                  >
                    <v-img
                      id="lazy-load-poster"
                      ref="modelViewerPoster"
                      slot="poster"
                      class="position-absolute"
                      :style="{ backgroundImage: 'url(' + `${gractorBaseUrl}/api/media/download?path=${item.properties.thumbnailPath}` + ')' }"
                      @mouseover="
                        () => {
                          captureAndSetPoster(index, 'card')
                        }
                      "
                    ></v-img
                    ><v-icon color="#FFD600" style="position: absolute; right: 0" :style="{ display: item.fav ? 'block' : 'none' }"
                      >mdi-star</v-icon
                    ></model-viewer
                  >
                </template>
                <template v-else>
                  <model-viewer
                    :ref="`modelViewer-${index}-card`"
                    :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.path}`"
                    style="display: block; width: 100%; height: 100%"
                    camera-controls
                    ><v-icon color="#FFD600" style="position: absolute; right: 0" :style="{ display: item.fav ? 'block' : 'none' }"
                      >mdi-star</v-icon
                    ></model-viewer
                  >
                </template></template
              >
              <template v-if="checkType(item.properties.mimetype) === 'photo'">
                <v-img
                  class="modelview-card-viewer-img"
                  :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.thumbnailPath}`"
                  style="width: 100%; height: 100%; border-radius: 0.1875rem"
                />
                <div style="font-size: 0.8125rem">{{ item.text }}</div>
              </template>
              <template v-if="checkType(item.properties.mimetype) === 'video'">
                <!-- <v-img class="modelview-card-viewer-img" :src="item.thumbnailname" style="width: 100%; height: 100%; border-radius: 0.1875rem" /> -->
                <iframe
                  width="100%"
                  height="100%"
                  :src="item.properties.srcPath + '?autoplay=1'"
                  frameborder="0"
                  allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
                  allowfullscreen
                ></iframe>
                <div style="font-size: 0.8125rem">{{ item.text }}</div>
                <div>{{ calVideoTime(item) }}</div>
              </template>
            </div>

            <p class="m-0 modelview-card-viewer-text d-flex" style="padding-left: 0.81rem !important">{{ truncateFilename(item.properties.name) }}</p>
          </v-list-item>
        </v-list>
      </div>
    </div>
  </section>
</template>

<script>
import TypecheckMixins from '~/components/viewtrack/source/divide-types-mixin'

export default {
  mixins: [TypecheckMixins],
  props: {
    isGrid: { type: Boolean, default: true },
    downloadModelingList: { type: Array, default: () => [] },
    selectedTagList: { type: Object, default: () => {} },
  },
  data() {
    return {}
  },
  computed: {
    darkMode() {
      return this.$vuetify.theme.dark
    },
    gractorBaseUrl() {
      return this.$config.viewtrack.gractorBaseUrl
    },
    productBaseUrl() {
      return this.$config.viewtrack.productBaseUrl
    },
  },
  methods: {
    clickComponent(item) {
      this.$emit('click-component', item)
    },
    truncateFilename(filename) {
      const maxLength = 10
      if (filename?.length > maxLength) {
        return filename.slice(0, maxLength) + '...'
      }
      return filename
    },
    checkModalTagIn(item, getOptionList) {
      if (Object.keys(getOptionList).length === 0) return true
      if (item.properties.tag) {
        const optionValues = Object.values(getOptionList).reduce((acc, option) => {
          if (option.label) {
            acc.push(option.label)
          }
          return acc
        }, [])
        const hasCommonTags = item.properties.tag.some(tag => optionValues.includes(tag))
        return hasCommonTags
      } else {
        return true
      }
    },
    async captureAndSetPoster(index, getType) {
      const modelViewer = this.$refs[`modelViewer-${index}-${getType}`][0]
      try {
        if (modelViewer) {
          modelViewer.reveal = 'auto'
          await this.waitForModelToLoad(modelViewer)
          modelViewer.dismissPoster()
        }
      } catch (error) {
        console.error('Error capturing the model image:', error)
      }
    },
    waitForModelToLoad(modelViewer) {
      return new Promise((resolve, reject) => {
        modelViewer.addEventListener('load', resolve, { once: true })
      })
    },
  },
}
</script>
<style>
.modelview-grid {
  column-gap: 1rem;
  row-gap: 1rem;
}
#lazy-load-poster {
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  background-color: #000;
  border-radius: 0.35714rem;
}

.modelview-card-viewer-img {
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  background-color: white;
}
.modelview-card-viewer-text {
  padding-left: 0.63rem !important;
  padding-right: 0.63rem !important;
  text-align: center;
  height: 3.19rem;
  line-height: normal;
  overflow: hidden;
  align-items: center;
}
</style>
