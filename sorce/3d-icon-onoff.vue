<template>
  <section>
    <template v-if="waitVersion">
      <v-btn :style="{ width: getSize + 'rem', height: getSize + 'rem' }" depressed icon @click="btnClick"
        ><v-icon style="width: 14px; height: 14px" :class="!isMapping ? firstIcon : secondIcon"></v-icon
      ></v-btn>
    </template>
    <template v-else>
      <v-btn :style="{ width: getSize + 'rem', height: getSize + 'rem' }" depressed icon @click="btnClick"
        ><v-icon :class="isOn ? firstIcon : secondIcon"></v-icon
      ></v-btn>
    </template>
  </section>
</template>

<script>
export default {
  components: {},
  props: {
    firstIcon: { type: String, default: '' },
    secondIcon: { type: String, default: '' },
    firstValue: { type: Boolean, default: true },
    waitVersion: { type: Boolean, default: false },
    isMapping: { type: Boolean, default: false },
    getSize: { type: Number, default: 3 },
  },
  data() {
    return {
      isOn: true,
    }
  },
  watch: {
    firstValue: {
      immediate: true,
      handler(newVal) {
        this.isOn = newVal
      },
    },
    isOn: {
      handler(newVal) {
        // console.info(newVal)
      },
    },
  },
  methods: {
    btnClick() {
      this.isOn = !this.isOn
      this.$emit('send-ison', this.isOn)
    },
  },
}
</script>
