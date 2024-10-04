<template>
  <div>
    <div class="d-flex" style="justify-content: space-between">
      <p style="margin-bottom: 0.88rem !important; padding: 0.38rem 0rem">{{ title }}</p>
      <v-btn ref="triangle-btn" icon depressed @click.stop="triangleDropdown"><v-icon :class="isdown ? 'dti-drop-down' : 'dti-drop-up'" small></v-icon></v-btn>
    </div>
    <div class="chip-display" :style="{ display: isdown ? 'flex' : 'none' }">
      <v-chip
        v-for="option in dataList"
        :key="option.value"
        class="chip-set"
        :class="{
          'active-solution': isColor && Object.keys(selectedOption).includes(option.value),
          'active-tag': !isColor && Object.keys(selectedOption).includes(option.value),
          'other-chip-dark': darkMode && isColor,
          'other-chip': !darkMode && isColor,
          'tag-chip': !isColor,
        }"
        @click.stop="totalSelectOption(option)"
      >
        {{ option.label }}
      </v-chip>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    title: { type: String, default: '' },
    isColor: { type: Boolean, default: true },
    dataList: { type: Array, default: () => [] },
    selectedOption: { type: Object, default: () => {} },
  },
  data() {
    return {
      isdown: false,
    }
  },
  computed: {
    darkMode() {
      return this.$vuetify.theme.dark
    },
  },
  watch: {
    selectedOption(newVal) {
      this.$emit('selected-option', newVal)
    },
  },
  methods: {
    triangleDropdown() {
      this.isdown = !this.isdown
    },
    totalSelectOption(option) {
      if (this.selectedOption[option.value]) {
        this.$delete(this.selectedOption, option.value)
      } else {
        this.$set(this.selectedOption, option.value, {
          label: option.label,
          value: option.value,
        })
      }
    },
  },
}
</script>
<style>
.active-solution {
  background-color: var(--cms-gray-003, rgba(126, 115, 254, 0.7)) !important;
  color: #fff !important;
}
.active-tag {
  border: 1.5px solid var(--cms-gray-004, rgba(126, 115, 254, 1)) !important;
}
.tag-chip {
  background-color: var(--cms-gray-003, #fff) !important;
  border: 1.5px solid var(--cms-gray-004, #f3f4f5);
}
.chip-display {
  display: flex;
  flex-wrap: wrap;
  gap: 0.35rem;
}
</style>
