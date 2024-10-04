<template>
  <div>
    <div></div>
    <div>
      <TemplateFormFieldVuetify
        :custom-field="true"
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
        @input="selectani"
      />
      <TemplateFormFieldVuetify
        :custom-field="true"
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
        @input="selectani"
      />
      <TemplateFormFieldVuetify
        :custom-field="true"
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
        @input="selectani"
      />
    </div>
    <div></div>
  </div>
</template>

<script>
export default {
  components: {
    TemplateFormFieldVuetify: () => import('~/components/template/template-form-field-vuetify.vue'),
  },
  data() {
    return {
      selectani: '',
    }
  },
}
</script>
