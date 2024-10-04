<template>
  <div>
    <div
      class="builder-side-box-taglist"
      style="position: relative; overflow: hidden; height: 3rem"
      :style="{ height: Object.keys(selectTagArray).length > 0 ? '3rem' : '0rem' }"
    >
      <v-chip-group ref="tagListContainer" active-class="''" column style="height: 100%">
        <v-chip
          v-for="tag in selectTagArray"
          :key="tag.value"
          class="select-chip"
          :close="isNeedClose"
          :outlined="tag.status === 'tag'"
          :color="isNeedClose ? '#EDEEEF' : '#f8f8f9'"
          @click:close="selectTagOption(tag)"
        >
          {{ tag.label || tag }}
        </v-chip>
      </v-chip-group>
      <v-btn
        icon
        :style="{ display: isTagOverflow ? 'block' : 'none' }"
        @click="
          {
            taglistOpen = !taglistOpen
          }
        "
        ><v-icon>mdi-dots-horizontal</v-icon></v-btn
      >
    </div>
    <div class="tag-box" :style="{ display: taglistOpen ? 'block' : 'none' }">
      <p style="margin-bottom: 0.64rem">선택한 태그</p>
      <v-chip-group ref="tagListContainer" active-class="''" column>
        <v-chip
          v-for="tag in selectTagArray"
          :key="tag.value"
          class="select-chip"
          :close="isNeedClose"
          :outlined="tag.status === 'tag'"
          color="#EDEEEF"
          @click:close="selectTagOption(tag)"
        >
          {{ tag.label || tag }}
        </v-chip>
      </v-chip-group>
      <div class="w-100" style="position: relative">
        <p
          class="tag-blox-close-btn mb-0"
          style="font-size: 1rem"
          @click="
            {
              taglistOpen = false
            }
          "
        >
          접기
        </p>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    selectTagArray: {
      type: [Array, Object],
      default: () => [],
    },
    isNeedClose: { type: Boolean, default: false },
  },
  data() {
    return {
      taglistOpen: false,
      isTagOverflow: false,
    }
  },
  watch: {
    selectTagArray(newVal, oldVal) {
      this.checkOverflow()
    },
  },
  created() {
    this.checkOverflow()
  },
  methods: {
    selectTagOption(option) {
      if (this.selectTagArray[option.value]) {
        this.$delete(this.selectTagArray, option.value)
      } else {
        this.$set(this.selectTagArray, option.value, {
          label: option.label,
          value: option.value,
        })
      }
    },
    checkOverflow() {
      if (Object.keys(this.selectTagArray).length > 2) {
        this.isTagOverflow = true
      } else {
        this.isTagOverflow = false
      }
    },
  },
}
</script>
<style>
.select-chip {
  padding: 0.36rem 0.86rem;
  font-size: 0.92857rem !important;
  color: #000;
}
</style>
