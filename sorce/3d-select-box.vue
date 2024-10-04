<template>
  <div style="position: relative" class="select-box">
    <div @click="toggleDropdown">
      <div
        class="custom-dropdown w-100 d-flex align-items-center justify-content-between g-cursor-pointer"
        :style="{
          backgroundColor: darkMode ? (isOpen ? '#4b4b4b' : '#2A2A2A') : isOpen ? '#fff' : '#f9fafb',
          border: isOpen ? '1.8px solid var(--cms-purple-003, #7E73FE)' : 'none',
        }"
      >
        <div class="d-flex align-items-center" :style="{ display: !isFilterIcon && filterWord == '' ? 'none !important' : 'block' }">
          <template v-if="isFilterIcon">
            <v-icon :color="darkMode ? '#F0F0F0' : 'rgba(47, 47, 47, 1)'" small style="width: 2rem; height: 2rem">mdi-filter</v-icon>
          </template>
          <template v-else
            ><p>{{ filterWord }}</p></template
          >
          <div style="width: 0.1rem; height: 0.97743rem; background-color: #7e7e7e; margin-right: 1.57rem; margin-left: 0.79rem"></div>
        </div>
        <div class="d-flex align-items-center justify-content-between w-100" @click.stop="toggleDropdown">
          <div class="dropdown-selected">
            {{ selected.label || 'Select an option' }}
          </div>
          <v-icon>mdi-chevron-down</v-icon>
        </div>
      </div>
      <div style="position: relative">
        <div
          :ref="dropdownRef"
          class="dropdown-menu"
          :style="{
            display: isOpen ? 'block' : 'none !important',
            backgroundColor: darkMode ? '#1F1F1F' : '#fff',
            boxShadow: '0px 3px 7px 0px rgba(0, 0, 0, 0.15)',
          }"
        >
          <div
            v-for="option in filterStates"
            :key="option.value"
            class="dropdown-item d-flex g-cursor-pointer mb-2"
            :style="{
              backgroundColor: darkMode ? (selected.value === option.value ? '#252525' : '#1F1F1F') : selected.value === option.value ? '#F9FAFB' : '#ffffff',
            }"
            @click.stop="selectOption(option)"
          >
            <v-icon
              :style="{
                visibility: selected.value === option.value ? 'visible' : 'hidden',
              }"
              style="margin-right: 2.07rem; color: rgba(126, 115, 254, 1); width: 2rem; height: 2rem"
              >mdi-check</v-icon
            >
            <div style="font-size: 1rem; line-height: 1.57143rem">{{ option.label }}</div>
          </div>
          <v-divider
            style="background-color: var(--cms-gray-004, #f3f4f5); margin: 0.5rem 0rem 1.14rem 0rem"
            :style="{ display: !solutionOn && !typeOn && !tagOn ? 'none' : 'block' }"
          ></v-divider>
          <div style="padding-left: 1rem; padding-right: 1rem">
            <div style="margin-bottom: 1.43rem" :style="{ display: solutionOn ? 'block' : 'none' }">
              <TagTemplate :title="`솔루션`" :is-color="true" :data-list="solutionStates" :selected-option="selectedOption"></TagTemplate>
            </div>
            <div style="margin-bottom: 1.43rem" :style="{ display: typeOn ? 'block' : 'none' }">
              <TagTemplate :title="`타입`" :is-color="true" :data-list="typeStates" :selected-option="selectedOption"></TagTemplate>
            </div>
            <div :style="{ display: tagOn ? 'block' : 'none' }">
              <TagTemplate :title="`태그`" :is-color="false" :data-list="tagStates" :selected-option="selectedOption"></TagTemplate>
            </div>
          </div>
          <div style="height: 1.7rem" :style="{ display: solutionOn || typeOn || tagOn ? 'block' : 'none' }"></div>
        </div>
      </div>
    </div>
    <template v-if="useTagList">
      <TagList :select-tag-array="selectedOption" :is-need-close="true"></TagList>
    </template>
  </div>
</template>

<script>
export default {
  components: {
    TagList: () => import('~/components/viewtrack/source/3d-taglist.vue'),
    TagTemplate: () => import('~/components/viewtrack/source/tag-template.vue'),
  },
  props: {
    id: { type: String, required: true },
    isFilterIcon: { type: Boolean, default: false },
    filterWord: { type: String, default: '' },
    getWord: { type: String, default: '' },
    getList: { type: Array, default: () => [] },
    solutionOn: { type: Boolean, default: true },
    typeOn: { type: Boolean, default: true },
    tagOn: { type: Boolean, default: true },
    useTagList: { type: Boolean, default: false },
    getTagList: { type: Array, default: () => [] },
  },
  data() {
    return {
      dropdownRef: `dropdownMenu_${this.generateRandomId()}`,
      isOpen: false, // 드롭다운 열림 상태
      selected: { label: '전체보기', value: 'all' }, // 현재 선택된 옵션
      filterStates: [
        { label: '전체보기', value: 'all' },
        { label: '즐겨찾기', value: 'fav' },
      ],
      solutionStates: [
        { label: 'FSMS', value: 'FSMS' },
        { label: 'AMI', value: 'AMI' },
        { label: 'Mobility', value: 'Mobility' },
        { label: 'Smart home', value: 'Smart home' },
      ],
      tagStates: [],
      typeStates: [
        { label: '사진', value: '사진' },
        { label: '동영상', value: '동영상' },
        { label: '3D', value: '3d' },
      ],
      selectedOption: {},
      taglistOpen: false,
    }
  },
  computed: {
    darkMode() {
      return this.$vuetify.theme.dark
    },
  },
  watch: {
    selectedOption(newVal, oldVal) {
      this.$emit('selected-option', newVal)
    },

    getTagList(newValue) {
      const newTags = newValue?.flatMap(item =>
        Array.isArray(item?.properties?.tag)
          ? item.properties.tag.map(tag => ({
              label: tag,
              value: tag + this.generateRandomId(),
            }))
          : []
      )

      this.tagStates = newTags?.reduce(
        (acc, tag) => {
          if (!acc.some(t => t.label === tag.label)) {
            acc.push(tag)
          }
          return acc
        },
        [...this.tagStates]
      )
    },
  },
  created() {
    this.selected = Object.keys(this.getList).length
      ? this.getWord !== ''
        ? { label: this.getWord, value: ' ' }
        : { label: '전체보기', value: 'all' }
      : { label: '전체보기', value: 'all' }
    this.filterStates = Object.keys(this.getList).length
      ? this.getList
      : [
          { label: '전체보기', value: 'all' },
          { label: '즐겨찾기', value: 'fav' },
        ]
  },
  mounted() {
    document.addEventListener('click', this.closeDropdown)
    const newTags = this.getTagList?.flatMap(item =>
      Array.isArray(item?.properties?.tag)
        ? item.properties.tag.map(tag => ({
            label: tag,
            value: tag + this.generateRandomId(),
          }))
        : []
    )

    this.tagStates = newTags?.reduce(
      (acc, tag) => {
        if (!acc.some(t => t.label === tag.label)) {
          acc.push(tag)
        }
        return acc
      },
      [...this.tagStates]
    )
  },
  beforeDestroy() {
    document.removeEventListener('click', this.closeDropdown)
  },
  methods: {
    toggleDropdown() {
      this.isOpen = !this.isOpen
    },
    selectOption(option) {
      this.selected = option
      //this.isOpen = false // 옵션 선택 후 드롭다운 닫기
    },
    closeDropdown(event) {
      const hasTriangleClass = event?.target?.className && typeof event.target.className === 'string' && event.target.className.includes('triangle')

      if (hasTriangleClass) {
        this.isOpen = true
      } else if (this.$refs.dropdownRef && !this.$refs.dropdownRef.contains(event.target) && !hasTriangleClass) {
        this.isOpen = false
      }
    },
    generateRandomId() {
      return 'id-' + Math.random().toString(36).substr(2, 9) + '-' + Date.now()
    },

    selectTagOption(option) {
      if (this.selectedOption[option.value]) {
        this.$delete(this.selectedOption, option.value)
      } else {
        this.$set(this.selectedOption, option.value, {
          label: option.label,
          value: option.value,
        })
      }
    },
    computedSelectBoxStyle() {
      const baseStyle = {
        border: this.isOpen ? '1.8px solid var(--cms-purple-003, #7E73FE)' : 'none',
      }

      if (this.darkMode) {
        baseStyle.backgroundColor = this.isOpen ? '#fff' : '#f9fafb'
      } else {
        baseStyle.backgroundColor = this.isOpen ? '#fff' : '#f9fafb'
      }

      return baseStyle
    },
  },
}
</script>
<style>
.custom-dropdown {
  /* position: relative; */
  border: none;
  user-select: none;
  background-color: var(--cms-gray-003, #f9fafb);
  border-radius: 0.1875rem;
  height: 2.625rem;
  padding: 0.43rem 1rem 0.43rem 1.29rem;
  font-family: 'Noto Sans KR' !important;
  height: fit-content;
}
.dropdown-selected {
  font-size: 1rem;
  line-height: 1.57143rem;
  letter-spacing: -0.01rem;
}

.dropdown-menu {
  min-width: 100% !important;
  position: absolute;
  background-color: #fff;
  border: 1px solid #ccc;
  border-top: none;
  z-index: 1000;
  display: block !important;
  padding: 0.29rem !important;
  /* width: 18rem; */
  width: 100%;
  font-family: 'Noto Sans KR' !important;
  border-radius: 0.21429rem;
  background: #fff;
  box-shadow: 0px 3px 7px 0px rgba(0, 0, 0, 0.15);
}

.dropdown-item {
  padding: 0.43rem 1rem;
  transition: background-color 0.2s;
  font-size: 1rem !important;
}

.dropdown-item:hover {
  background-color: #f0f0f0;
}

.chip-set {
  padding: 0.29rem 1rem !important;
  text-align: center !important;
  font-size: 0.92857rem !important;
  width: fit-content;
  display: flex;
  justify-content: center;
}
.other-chip {
  background-color: #f9fafb !important;
}

.other-chip-dark {
  background-color: #2a2a2a !important;
}
.builder-side-box-taglist {
  overflow-y: auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.tag-box {
  box-shadow: 0px 3px 7px 0px rgba(0, 0, 0, 0.15);
  position: absolute;
  background-color: #fff;
  width: 100%;
  height: fit-content;
  padding: 0.62rem 0.88rem;
  z-index: 10;
  top: 50%;
  left: 0;
  border-radius: 0.1875rem;
}
.tag-blox-close-btn {
  display: flex;
  justify-content: flex-end;
  cursor: pointer;
  color: #554ad3;
  font-size: 0.8125rem;
  padding: 0.25rem 0rem;
}
.tag-box .transparent-bg {
  background-color: #ffffff !important;
}

.tag-box .default-bg {
  background-color: #edeeef !important;
}
/* .select-box p,
.select-box span,
.select-box div {
  color: #1e1e1e !important;
} */
</style>
