<template>
  <div>
    <div :style="{ display: !clickSearch ? 'block' : 'none' }" style="padding: 0rem 0.94rem">
      <Selectbox3d
        id="selectbox1"
        :is-filter-icon="true"
        :solution-on="true"
        :type-on="true"
        :tag-on="true"
        :use-tag-list="true"
        :get-tag-list="downloadModelingList"
        @selected-option="getSelectOption"
      ></Selectbox3d>
    </div>
    <div
      class="modelview-group"
      style="flex: 1; display: flex; flex-direction: column"
      :style="{ display: !clickSearch ? 'flex' : inputSearchCheck ? 'flex' : 'none' }"
    >
      <div class="d-flex align-items-center justify-content-between" style="padding: 1.43rem 1.07rem 1rem 1.07rem">
        <v-checkbox
          v-model="favBtn"
          class="fav_btn"
          dense
          style="margin-top: 0 !important; padding-left: 0.8rem; font-size: 1rem"
          :label="`즐겨찾기`"
        ></v-checkbox>
        <v-btn-toggle v-model="sideboxGridOn" style="gap: 0.5rem" mandatory>
          <v-btn style="height: 2rem; min-width: 2rem" value="layer" class="list-menu-btn">
            <v-icon :color="sideboxGridOn === 'layer' ? 'white' : 'rgba(155, 156, 157, 1)'" small>mdi-format-list-bulleted</v-icon>
          </v-btn>

          <v-btn style="height: 2rem; min-width: 2rem" value="grid" class="list-menu-btn"
            ><v-icon small :color="sideboxGridOn === 'grid' ? 'white' : 'rgba(155, 156, 157, 1)'">mdi-view-grid</v-icon></v-btn
          ></v-btn-toggle
        >
      </div>
      <div class="modelview">
        <MediaListCard
          :is-grid="sideboxGridOn === 'grid'"
          :download-modeling-list="downloadModelingList"
          :selected-tag-list="getMediaSelectedOption"
          @click-component="clickComponent"
        ></MediaListCard>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  components: {
    MediaListCard: () => import('~/components/viewtrack/source/media-list-card.vue'),
    Selectbox3d: () => import('~/components/viewtrack/source/3d-select-box.vue'),
  },
  props: {
    clickSearch: { type: Boolean, default: false },
    downloadModelingList: { type: Array, default: () => [] },
    inputSearchCheck: { type: Boolean, default: false },
  },
  data() {
    return {
      sideboxGridOn: 'grid',
      getMediaSelectedOption: {},
      favBtn: false,
    }
  },
  methods: {
    getSelectOption(data) {
      this.getMediaSelectedOption = data
    },
    clickComponent(item) {
      this.$emit('click-component', item)
    },
  },
}
</script>

<style>
.list-menu-btn {
  border: 0px !important;
  padding: 0 0 !important;
  border-radius: 0.23rem !important;
}
@media (max-width: 2560px) {
  .modelview-group {
    max-height: calc(9.1rem * 5 + 0.5rem * 5);
  }
}
@media (max-width: 1920px) {
  .modelview-group {
    max-height: calc(9.1rem * 5 + 0.5rem * 5);
  }
}
.modelview {
  overflow-x: hidden;
  overflow-y: auto;
  /* max-height: 70vh; */
  position: relative;
  padding: 0rem 1rem 1rem 1rem;
  margin-right: 0.5rem;
  display: flex;
}
.fav_btn .v-messages {
  min-height: 0 !important;
}
</style>
