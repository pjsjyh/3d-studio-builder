<template>
  <div class="justify-content-center d-flex flex-column align-items-center" style="width: 22.57rem">
    <div class="d-flex" style="gap: 0.71rem">
      <TemplateFormFieldVuetify
        :value="searchQuery"
        class="mt-3"
        :custom-field="true"
        :field-model="{
          type: 'text',
        }"
        :bind-value="{
          'hide-details': true,
          dense: true,
          outlined: true,
          label: '오프셋 X',
          clearable: true,
          color: 'deep-purple accent-2',
          'show-size': true,
          'truncate-length': 15,
        }"
      />
      <TemplateFormFieldVuetify
        :value="searchQuery"
        class="mt-3"
        :custom-field="true"
        :field-model="{
          type: 'text',
        }"
        :bind-value="{
          'hide-details': true,
          dense: true,
          outlined: true,
          label: '오프셋 Y',
          clearable: true,
          color: 'deep-purple accent-2',
          'show-size': true,
          'truncate-length': 15,
        }"
      />
    </div>
    <v-radio-group v-model="radioGroup" class="w-100" mandatory>
      <v-radio key="1" value="1" class="custom-radio" style="align-items: flex-start">
        <template #label>
          <div style="gap: 0.93rem; display: grid">
            <div>없음</div>
          </div>
        </template></v-radio
      >
      <v-radio key="2" value="2" label="아이콘" class="custom-radio" style="align-items: flex-start">
        <template #label>
          <div>
            <div>아이콘</div>
            <div :class="radioClass" style="gap: 0.93rem">
              <div class="d-flex align-items-center justify-content-between">
                <TemplateFormFieldVuetify
                  :value="iconURL"
                  :custom-field="true"
                  :field-model="{
                    type: 'text',
                  }"
                  :bind-value="{
                    'hide-details': true,
                    dense: true,
                    outlined: true,
                    label: '아이콘',
                    clearable: true,
                    color: 'deep-purple accent-2',
                    'show-size': true,
                    'truncate-length': 15,
                  }"
                />
                <v-btn depressed icon style="width: 2.85714rem; height: 2.85714rem; background-color: var(--DT-light-dt_bg_3, #f8f8f9)"
                  ><v-icon large class="dti-upload" style="width: 2.85714rem; height: 2.85714rem"></v-icon
                ></v-btn>
              </div>
              <TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'select',
                  items: iconSizeOption,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  placeholder: '위치',
                  color: 'deep-purple accent-2',
                }"
              />
              <TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'select',
                  items: iconSizeOption,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  placeholder: '크기',
                  color: 'deep-purple accent-2',
                }"
              />
              <TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'select',
                  items: iconBackgroundOption,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  placeholder: '배경',
                  color: 'deep-purple accent-2',
                }"
              />
              <div class="d-flex">
                <TemplateFormFieldVuetify
                  :value="searchQuery"
                  class="mt-3"
                  :custom-field="true"
                  :field-model="{
                    type: 'text',
                  }"
                  :bind-value="{
                    'hide-details': true,
                    dense: true,
                    outlined: true,
                    label: '배경 색상',
                    clearable: true,
                    color: 'deep-purple accent-2',
                    'show-size': true,
                    'truncate-length': 15,
                  }"
                />
                <TemplateFormFieldVuetify
                  :value="searchQuery"
                  class="mt-3"
                  :custom-field="true"
                  :field-model="{
                    type: 'text',
                  }"
                  :bind-value="{
                    'hide-details': true,
                    dense: true,
                    outlined: true,
                    label: '불투명도',
                    clearable: true,
                    color: 'deep-purple accent-2',
                    'show-size': true,
                    'truncate-length': 15,
                  }"
                />
              </div>
            </div>
          </div> </template
      ></v-radio>
      <v-radio key="3" value="3" class="custom-radio"> <template #label> ㅇㅇㅇ </template></v-radio>
    </v-radio-group>
  </div>
</template>

<script>
export default {
  components: {
    TemplateFormFieldVuetify: () => import('~/components/template/template-form-field-vuetify.vue'),
  },
  data() {
    return {
      radioGroup: 1,
      searchQuery: '',
      mediaSrcFilterOption: [
        { text: '반복 없음', value: 'no' },
        { text: '사용자 지정', value: 'user' },
      ],
      iconURL: '',
      iconSize: '',
      iconSizeOption: [
        { text: '원본 유지', value: 'm' },
        { text: '크게', value: 'l' },
        { text: '작게', value: 's' },
      ],
      iconBackground: '',
      iconBackgroundOption: [
        { text: '사각형', value: 'square' },
        { text: '원형', value: 'round' },
      ],
      radioClass: 'hide-grid',
    }
  },
  watch: {
    radioGroup(newVal) {
      if (newVal != 2) {
        this.radioClass = 'hide-grid'
      } else {
        this.radioClass = 'show-grid'
      }
    },
  },
}
</script>

<style>
.show-grid {
  display: grid !important;
}

.hide-grid {
  display: none !important;
}
.custom-radio .v-input--selection-controls__input {
  transform: scale(0.75); /* 라디오 버튼 크기 줄이기 */
}
/* 
.custom-radio .v-input--selection-controls__ripple {
  width: 1rem;
  height: 1rem; 
} */

.custom-radio .v-input--selection-controls__label {
  font-size: 14px; /* 라벨 텍스트 크기 줄이기 */
}
</style>
