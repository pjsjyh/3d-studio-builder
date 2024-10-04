<template>
  <div class="d-flex flex-column" style="flex: 1">
    <!-- <div class="d-flex justify-content-between" style="margin-bottom: 1.14rem">
      <v-btn-toggle v-model="animationSelect" class="w-100" tile color="accent-3 group" active-class="animation-btn-active" style="gap: 1.43rem" mandatory>
        <v-btn
          class="animation-btn"
          value="mesh"
          :style="{
            fontWeight: animationSelect === 'mesh' ? '600' : '400',
          }"
          >메시 애니메이션</v-btn
        >
        <v-btn
          class="animation-btn"
          value="scene"
          :style="{
            fontWeight: animationSelect === 'scene' ? '600' : '400',
          }"
          >씬 애니메이션</v-btn
        >
      </v-btn-toggle>
    </div> -->

    <div v-show="animationSelect === 'scene'">
      <v-btn-toggle
        v-model="animationTypeSelect"
        class="w-100"
        tile
        color="accent-3 group"
        active-class="animation-btn-active"
        style="gap: 0.71rem; margin-bottom: 0.64rem"
        mandatory
      >
        <v-btn
          class="animation-btn-small"
          value="mesh"
          style="font-size: 0.85714rem"
          :style="{
            fontWeight: animationSelect === 'mesh' ? '600' : '400',
          }"
          >메시</v-btn
        >
        <v-btn
          class="animation-btn-small"
          value="camera"
          style="font-size: 0.85714rem"
          :style="{
            fontWeight: animationSelect === 'scene' ? '600' : '400',
          }"
          >카메라</v-btn
        >
        <v-btn
          class="animation-btn-small"
          value="light"
          style="font-size: 0.85714rem"
          :style="{
            fontWeight: animationSelect === 'scene' ? '600' : '400',
          }"
          >빛</v-btn
        >
      </v-btn-toggle>
      <template v-if="animationTypeSelect === 'mesh'">
        <v-btn-toggle v-model="meshSelect" tile color="deep-purple accent-3" style="gap: 0.57rem; margin-top: 1.43rem" group>
          <v-btn class="lightSelectBox" value="meshmove"> <v-icon class="lightIcon dti-move"></v-icon><span>움직임</span> </v-btn>
          <v-btn class="lightSelectBox" value="meshmaterial"> <v-icon class="lightIcon dti-material"></v-icon><span>머티리얼</span> </v-btn>
        </v-btn-toggle>
        <template v-if="meshSelect === 'meshmove'">
          <div class="d-flex justify-content-between">
            <v-btn depressed icon style="width: fit-content" @click="maekMeshMove"><v-icon small>mdi-plus</v-icon>지점 추가</v-btn>
          </div>
          <div style="overflow-y: auto">
            <EventRules v-for="(item, index) in sceneMeshMove" :key="index"> </EventRules>
          </div>
        </template>
        <template v-if="meshSelect === 'meshmaterial'">
          <div class="d-flex justify-content-between">
            <span style="color: #ff7373; font-size: 0.85714rem">* 메쉬를 선택하지 않아도 됩니다.</span>
          </div>

          <v-text-field label="Outlined" outlined></v-text-field> </template
      ></template>
      <template v-if="animationTypeSelect === 'camera'">
        <v-btn-toggle v-model="cameraSelect" tile color="deep-purple accent-3" style="gap: 0.57rem; margin-top: 1.43rem" group>
          <v-btn class="lightSelectBox" value="homcam"> <v-icon class="lightIcon dti-cam-home"></v-icon><span>홈 포지션</span> </v-btn>
          <v-btn class="lightSelectBox" value="meshcam"> <v-icon class="lightIcon dti-cam-position"></v-icon><span>메시 포지션</span> </v-btn>
        </v-btn-toggle>
        <template v-if="cameraSelect === 'homcam'">
          <div class="d-flex justify-content-between">
            <span style="color: #ff7373; font-size: 0.85714rem">* 메쉬를 선택하지 않아도 됩니다.</span>
            <v-btn depressed icon style="width: fit-content" @click="makeCameraHomeAni"><v-icon small>mdi-plus</v-icon>카메라 위치 추가</v-btn>
          </div>

          <v-text-field label="Outlined" outlined></v-text-field>
        </template>
        <template v-else-if="cameraSelect === 'meshcam'">
          <v-btn depressed icon style="width: fit-content" @click="makeMeshAni"><v-icon small>mdi-plus</v-icon>애니메이션 추가</v-btn>

          <v-text-field label="Outlined" outlined></v-text-field> </template
      ></template>
      <template v-if="animationTypeSelect === 'light'">
        <v-btn-toggle v-model="lightSelect" tile color="deep-purple accent-3" style="gap: 0.57rem; margin-top: 1.43rem" group mandatory>
          <v-btn class="lightSelectBox" value="pointLight" @click="lightSelectClick('pointLight')">
            <v-icon class="lightIcon dti-light-point"></v-icon><span>포인트</span>
          </v-btn>

          <v-btn class="lightSelectBox" value="spotLight" @click="lightSelectClick('spotLight')">
            <v-icon class="lightIcon dti-light-spotlight"></v-icon><span>스포트라이트</span>
          </v-btn>

          <v-btn class="lightSelectBox" value="directionLight" @click="lightSelectClick('directionLight')">
            <v-icon class="lightIcon dti-light-direction"></v-icon><span>방향성조명</span>
          </v-btn>

          <v-btn class="lightSelectBox" value="domeLight" @click="lightSelectClick('domeLight')">
            <v-icon class="lightIcon dti-light-hemisphere"></v-icon><span>반구형빛</span>
          </v-btn>
          <v-btn style="display: none" value="fake"></v-btn>
        </v-btn-toggle>
        <template v-if="lightSelect != 'fake'">
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
                  $set(moveVal, 'posX', value)
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
                  $set(moveVal, 'posY', value)
                }
              "
            /><TemplateFormFieldVuetify
              :custom-field="true"
              :value="moveVal.posZ"
              :field-model="{
                type: 'text',
                items: setOriginalAni,
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
                  $set(moveVal, 'posZ', value)
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
                items: setOriginalAni,
              }"
              :bind-value="{
                clearable: true,
                'hide-details': true,
                dense: true,
                outlined: true,
                label: '각도 X',
                color: 'deep-purple accent-2',
                disabled: lightSelect === 'spotLight' ? false : true,
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
                items: setOriginalAni,
              }"
              :bind-value="{
                clearable: true,
                'hide-details': true,
                dense: true,
                outlined: true,
                label: '각도 Y',
                color: 'deep-purple accent-2',
                disabled: lightSelect === 'spotLight' ? false : true,
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
                items: setOriginalAni,
              }"
              :bind-value="{
                clearable: true,
                'hide-details': true,
                dense: true,
                outlined: true,
                label: '각도 Z',
                color: 'deep-purple accent-2',
                disabled: lightSelect === 'spotLight' ? false : true,
              }"
              @input="
                value => {
                  moveVal.rotZ = value
                  $set(moveVal, 'rotZ', value)
                }
              "
            />
          </div>
          <div style="margin-top: 3.14rem">
            <div class="w-100 justify-content-start g-cursor-pointer" style="padding: 0.29rem 0rem">
              <v-icon style="margin-right: 0.79rem; width: 2rem; height: 2rem">mdi-menu-down</v-icon><span>그림자</span>
            </div>
            <div class="d-flex" style="gap: 0.57rem">
              <TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'combo',
                  items: setOriginalAni,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  label: '부드럽게',
                  color: 'deep-purple accent-2',
                }"
                @input="selectani"
              /><TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'combo',
                  items: setOriginalAni,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  label: '바이어스',
                  color: 'deep-purple accent-2',
                }"
                @input="selectani"
              />
            </div>
            <div class="d-flex" style="gap: 0.57rem; margin-top: 1.14rem">
              <TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'combo',
                  items: setOriginalAni,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  label: '오프셋',
                  color: 'deep-purple accent-2',
                }"
                @input="selectani"
              /><TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'combo',
                  items: setOriginalAni,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  label: '품질',
                  color: 'deep-purple accent-2',
                }"
                @input="selectani"
              />
            </div>
          </div>
          <div style="margin-top: 3.14rem">
            <div class="w-100 justify-content-start g-cursor-pointer" style="padding: 0.29rem 0rem">
              <v-icon style="margin-right: 0.79rem; width: 2rem; height: 2rem">mdi-menu-down</v-icon><span>조명 옵션</span>
            </div>
            <div class="d-flex" style="gap: 0.57rem">
              <TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'combo',
                  items: setOriginalAni,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  label: '부드럽게',
                  color: 'deep-purple accent-2',
                }"
                @input="selectani"
              /><TemplateFormFieldVuetify
                :custom-field="true"
                :field-model="{
                  type: 'combo',
                  items: setOriginalAni,
                }"
                :bind-value="{
                  clearable: true,
                  'hide-details': true,
                  dense: true,
                  outlined: true,
                  label: '밝기',
                  color: 'deep-purple accent-2',
                }"
                @input="selectani"
              />
            </div>
          </div>
          <v-btn class="w-100" style="margin-top: 4rem">추가</v-btn>
        </template>
        <template v-else>
          <MediaListEmpty
            style="margin-top: 3rem"
            :get-word="'매쉬 데이터 매핑 리스트에서 선택한 후\n 씬 애니메이션을 제작해 보세요.\n\n * 포인트 / 방향성 조명 제외 '"
            :get-img="'dti-component'"
            :img-size="'3rem'"
            :is-visible="false"
          ></MediaListEmpty> </template
      ></template>
    </div>
  </div>
</template>

<!-- @click:append="sdfs" -->
<script>
export default {
  components: {
    IconOnOff: () => import('~/components/viewtrack/source/3d-icon-onoff.vue'),
    MediaListEmpty: () => import('~/components/viewtrack/source/media-list-empty.vue'),
    EventRules: () => import('~/components/viewtrack/source/3d-make-event-rules.vue'),
    TemplateFormFieldVuetify: () => import('~/components/template/template-form-field-vuetify.vue'),
  },
  props: {
    settingModelingList: { type: Array, default: () => [] },
    settingModelingAniName: { type: Array, default: () => [] },
    aniMappingClickNum: { type: Array, default: () => [] },
    objectAniClickItem: {
      type: Object,
      default: () => {},
    },
  },
  data() {
    return {
      animationSelect: 'scene',
      makedMeshAnimationList: {},
      animationCount: 0,
      setOriginalAni: [],
      selectani: '',
      tab: null,

      meshAniList: [],

      animationTypeSelect: 'mesh',
      sceneAniList: [],
      sceneSettingAni: {}, //여기안에 type, properties있음 현재 셋팅중인 씬 애니
      sceneAniProperties: {},

      lightSelect: 'fake',
      meshSelect: '',
      cameraSelect: '',
      moveVal: {
        posX: 0,
        posY: 0,
        posZ: 0,
        rotX: 0,
        rotY: 0,
        rotZ: 0,
      },
      sceneMeshMove: [],
    }
  },
  watch: {
    objectAniClickItem(newVal) {
      this.sceneAniSave()
      if (newVal.animationType.toLowerCase().includes('light')) {
        this.animationTypeSelect = 'light'
        this.moveVal = newVal.sceneAniProperties
      } else if (newVal.animationType.toLowerCase().includes('camera')) {
        this.animationTypeSelect = 'camera'
      } else if (newVal.animationType.toLowerCase().includes('mesh')) {
        this.animationTypeSelect = 'mesh'
      } else {
        this.resetMoveVal()
      }
      this.sceneSettingAni = newVal
    },
    // moveVal: {
    //   handler(newVal) {
    //     console.info(newVal.posX)
    //   },
    //   deep: true,
    // },
    ///this.$set(this.meshAniList[index].moveVal, type, value);
    //this.moveVal
    // settingModelingList(newVal) {
    //   this.meshAniList = newVal
    //   this.makedMeshAnimationList = newVal
    //      :setting-modeling-list="settingModelingList"
    //
    // },
    // makedMeshAnimationList(newVal) {
    //   // console.info(newVal)
    //   // this.$emit('make-ani', this.makedMeshAnimationList)
    // },
  },
  methods: {
    resetMoveVal() {
      this.moveVal.posX = 0
      this.moveVal.posY = 0
      this.moveVal.posZ = 0
      this.moveVal.rotX = 0
      this.moveVal.rotY = 0
      this.moveVal.rotZ = 0
    },
    generateRandomId() {
      return 'sceneAni-' + Math.random().toString(36).substr(2, 9) + '-' + Date.now()
    },
    lightSelectClick(newVal) {
      this.sceneAniSave()
      //this.resetMoveVal()
      if (newVal != 'spotLight') {
        this.moveVal.posX = 0
        this.moveVal.posY = 0
        this.moveVal.posZ = 0
        this.moveVal.rotX = 0
        this.moveVal.rotY = 0
        this.moveVal.rotZ = 0
      }
      this.sceneSettingAni = {}
      this.sceneSettingAni.sceneAniTempatedId = this.generateRandomId()
      this.sceneSettingAni.animationType = newVal
      this.sceneSettingAni.sceneAniProperties = { ...this.moveVal }
      this.sceneAniList.push(this.sceneSettingAni)
      this.$emit('now-setting-scene-ani', this.sceneSettingAni)
      this.$emit('make-scene-ani', this.sceneAniList)
    },
    sceneAniSave() {
      this.sceneSettingAni.sceneAniProperties = { ...this.moveVal }

      if (Object.keys(this.sceneAniList).length != 0) {
        const isIn = this.sceneAniList.find(ani => ani.sceneAniTempatedId === this.sceneSettingAni.sceneAniTempatedId)
        if (isIn) {
          const index = this.sceneAniList.indexOf(isIn)
          if (index !== -1) {
            this.$set(this.sceneAniList, index, {
              ...isIn,
              ...this.sceneSettingAni,
            })
          }
        } else {
          this.sceneAniList.push(this.sceneSettingAni)
        }
      }
    },
    test(getkey) {
      for (let i = this.sceneAniList.length - 1; i >= 0; i--) {
        if (this.sceneAniList[i].sceneAniTempatedId === getkey) {
          this.sceneAniList.splice(i, 1)
        }
      }
    },
    setEyeValue(value, index) {
      // item.disable[0] = value
      this.meshAniList[index].disable[0] = value
      this.$set(this.meshAniList[index].disable, 0, value)
      this.$emit('make-ani', this.meshAniList)
    },
    handleAppendIconClick() {
      // console.info('Append icon clicked!')
      // 아이콘 클릭 시 처리할 로직
    },
    selectList(newVal, key) {
      if (Object.prototype.hasOwnProperty.call(this.makedMeshAnimationList, key)) {
        this.makedMeshAnimationList[key] = {
          ...this.makedMeshAnimationList[key],
          ...newVal,
        }
        // this.$emit('make-ani', this.makedMeshAnimationList)
      }
    },
    getAniSetting(newVal) {},
    makeMeshAni() {
      const uniqueId = 'meshani' + Date.now() + Math.random().toString(16).slice(2)
      const pushitem = { animationIndex: 0, name: `애니메이션 ${this.animationCount}`, repeat: 0, delay: 0 }
      this.$set(this.makedMeshAnimationList, uniqueId, pushitem)
      this.animationCount += 1
      //this.$emit('make-ani', this.makedMeshAnimationList)
    },
    makeLightAni(type) {
      // if (type === 'point') {
      //   this.sceneSettingAni
      // } else if (type === 'spot') {
      // } else if (type === 'direction') {
      // } else if (type === 'dome') {
      // }
    },
    makeCameraHomeAni() {},
    maekMeshMove() {
      this.sceneMeshMove.push([])
    },
    removeMeshAni(eventId) {
      this.deleteKey(this.makedMeshAnimationList, eventId)
    },
    deleteKey(obj, keyToDelete) {
      for (const key in obj) {
        if (key === keyToDelete) {
          this.$delete(obj, key)
        } else if (typeof obj[key] === 'object' && obj[key] !== null) {
          this.deleteKey(obj[key], keyToDelete)
        }
      }
    },
    setMeshAni(getList) {
      this.makedMeshAnimationList = getList
      // console.info(this.makedMeshAnimationList)
    },
    deleteAniList(getId) {
      if (Object.keys(this.sceneAniList).length != 0) {
        const isIn = this.sceneAniList.find(ani => ani.sceneAniTempatedId === getId)
        if (isIn) {
          const index = this.sceneAniList.indexOf(isIn)
          if (index !== -1) {
            // 해당 항목을 리스트에서 삭제
            this.sceneAniList.splice(index, 1)
          }
        }
      }
    },
  },
}
</script>

<style>
.input-btn {
  position: absolute;
  right: 5%;
  top: 50%;
  transform: translateY(-60%);
}
.animation-btn {
  flex: 1;
  text-align: center;
  padding-top: 0.71rem !important;
  padding-bottom: 0.71rem !important;
  border-radius: 3.05421rem !important;
  background-color: #fff !important;
  border: none !important;
  font-size: 1rem !important;
}
.animation-btn-active {
  color: rgba(85, 74, 211, 1) !important;
}
.animation-tab .v-tabs-bar {
  width: 100%;
}
.lightSelectBox {
  width: 5.5rem;
  margin: 0 !important;
  border-radius: 0.35714rem !important;
  border: 1.5px solid #f0f0f0 !important;
  padding-top: 0.71rem !important;
  padding-bottom: 0.86rem !important;
}
.lightSelectBox .v-btn__content {
  display: flex;
  flex-direction: column;
}
.lightIcon {
  width: 2.85714rem;
  height: 2.85714rem;
}
.sceneAni .v-list-item::after {
  min-height: 0 !important;
}
</style>
