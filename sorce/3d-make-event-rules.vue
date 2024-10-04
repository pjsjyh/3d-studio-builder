<template>
  <div class="w-100 event-rules mb-3">
    <div class="d-flex align-items-center justify-content-between mb-2 pl-3">
      지점
      <v-btn icon class="trash-icon" @click="emitDeleteEvent(getKey)"><v-icon>mdi-trash-can-outline</v-icon></v-btn>
    </div>
    <div class="d-flex" style="gap: 0.36rem; margin-top: 3.14rem">
      <TemplateFormFieldVuetify
        :custom-field="true"
        :value="moveVal.posX"
        :field-model="{
          type: 'text',
        }"
        :bind-value="{
          clearable: true,
          'hide-details': true,
          dense: true,
          outlined: true,
          label: '이동 X',
          color: 'deep-purple accent-2',
        }"
        @input="
          value => {
            moveVal.posX = value
          }
        "
      /><TemplateFormFieldVuetify
        :custom-field="true"
        :value="moveVal.posY"
        :field-model="{
          type: 'text',
        }"
        :bind-value="{
          clearable: true,
          'hide-details': true,
          dense: true,
          outlined: true,
          label: '이동 Y',
          color: 'deep-purple accent-2',
        }"
        @input="
          value => {
            moveVal.posY = value
          }
        "
      /><TemplateFormFieldVuetify
        :custom-field="true"
        :value="moveVal.posZ"
        :field-model="{
          type: 'text',
        }"
        :bind-value="{
          clearable: true,
          'hide-details': true,
          dense: true,
          outlined: true,
          label: '이동 Z',
          color: 'deep-purple accent-2',
        }"
        @input="
          value => {
            moveVal.posZ = value
          }
        "
      />
    </div>
    <div class="d-flex" style="gap: 0.36rem; margin-top: 1.86rem">
      <TemplateFormFieldVuetify
        :custom-field="true"
        :value="moveVal.rotX"
        :field-model="{
          type: 'text',
        }"
        :bind-value="{
          clearable: true,
          'hide-details': true,
          dense: true,
          outlined: true,
          label: '각도 X',
          color: 'deep-purple accent-2',
        }"
        @input="
          value => {
            moveVal.rotX = value
          }
        "
      /><TemplateFormFieldVuetify
        :custom-field="true"
        :value="moveVal.rotY"
        :field-model="{
          type: 'text',
        }"
        :bind-value="{
          clearable: true,
          'hide-details': true,
          dense: true,
          outlined: true,
          label: '각도 Y',
          color: 'deep-purple accent-2',
        }"
        @input="
          value => {
            moveVal.rotY = value
          }
        "
      /><TemplateFormFieldVuetify
        :custom-field="true"
        :value="moveVal.rotZ"
        :field-model="{
          type: 'text',
        }"
        :bind-value="{
          clearable: true,
          'hide-details': true,
          dense: true,
          outlined: true,
          label: '각도 Z',
          color: 'deep-purple accent-2',
        }"
        @input="
          value => {
            moveVal.rotZ = value
          }
        "
      />
    </div>
  </div>
</template>

<script>
export default {
  components: {
    TemplateFormFieldVuetify: () => import('~/components/template/template-form-field-vuetify.vue'),
  },
  props: {},
  data() {
    return {
      moveVal: {
        posX: 0,
        posY: 0,
        posZ: 0,
        rotX: 0,
        rotY: 0,
        rotZ: 0,
      },
    }
  },
  watch: {},
  methods: {},
}
</script>

<style>
.trash-icon {
  width: 2.44336rem;
  height: 2.4436rem;
}

.event-rules .v-text-field {
  padding: 0;
}
</style>
