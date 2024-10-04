<template>
  <v-combobox
    v-model="selected"
    :items="getList"
    persistent-hint
    :placeholder="getWord"
    class="combo-style"
    :class="{ 'combo-style-none-select': selected === getWord }"
    :style="{ width: getWidth }"
  ></v-combobox>
</template>

<script>
export default {
  props: {
    getList: { type: Array, default: () => [] },
    getWord: { type: String, default: '' },
    getWidth: { type: String, default: '100%' },
  },
  data() {
    return {
      isOpen: false, // 드롭다운 열림 상태
      selected: '',
    }
  },
  methods: {
    toggleDropdown() {
      this.isOpen = !this.isOpen
    },
    selectOption(option) {
      this.selected = option
      this.isOpen = false // 옵션 선택 후 드롭다운 닫기
    },
  },
}
</script>
<style>
.combo-style .v-input__slot {
  height: 2.8rem;
}

.combo-style .v-select__slot {
  padding-left: 0.5rem;
}

.combo-style-none-select .v-select__slot > input {
  color: #1e1e1e;
  opacity: 0.5;
}
.combo-style .v-input__slot:before {
  border-style: none;
}
</style>
