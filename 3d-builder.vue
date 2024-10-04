<template>
  <v-app class="v-app-class" style="overflow: hidden">
    <dt-sub-nav
      class="f-noshrink"
      home-link="/viewtrack/p/component"
      page="3D 빌더"
      :title="title"
      :tag="tagData"
      @addTag="handleAddTag"
      @input="inputTitle"
      @onClickBtn="onClickSubNavBtn"
    >
      <template v-if="isDialog" #goBack>
        <v-icon cursor @click="$emit('goBack')">dti-back</v-icon>
      </template>
    </dt-sub-nav>
    <div style="display: flex; flex-direction: column; flex: 1">
      <v-dialog v-model="finishDialog" style="width: 20% !important" max-width="27rem">
        <div class="finishDialog-main dialog-set-main bg-white d-flex flex-column justify-content-center align-items-center position-relative">
          <v-icon color="#FF6060" style="margin-bottom: 0.44rem; width: 2.625rem; height: 2.625rem">mdi-alert-circle</v-icon>
          <p class="m-0" style="text-align: center">
            홈으로 가기 전에 에 대한<br />
            변경 사항을 저장하시겠습니까?
          </p>
          <div class="finishDialog-line w-100" style="background-color: #f4f4f4; height: 0.1rem; margin-top: 1.93rem"></div>
          <div class="finishDialog-btn-group d-flex justify-content-end w-100" style="gap: 0.44rem; padding: 0.81rem 0.88rem">
            <v-btn depressed>취소</v-btn>
            <v-btn depressed>다른 이름으로 저장</v-btn>
            <v-btn depressed style="background-color: rgba(126, 115, 254, 1); color: white">저장</v-btn>
          </div>
        </div>
      </v-dialog>
      <div class="builder-class h-100 d-flex justify-content-between position-relative" :style="`${darkMode ? 'background: #2A2A2A' : 'background: #f8f8f9'}`">
        <div class="h-100 d-flex">
          <mediaList
            ref="mediaListClickCom"
            :setting-modeling-list="nowSettingModeling"
            :media-set="saveMediaSet"
            @now-setting-mapping="nowSettingMapping"
            @sidebox-selected="getsideboxSelected"
            @send-ison="sendIson"
            @send-lock-ison="sendLockIson"
            @resetting-moedling-list="resettingModelingList"
            @click-component="clickComponent"
            @now-click-temp="nowClickTemp"
          ></mediaList>
        </div>
        <div class="canvas-control-btn-group position-absolute translate-middle-x d-flex gap-3" style="bottom: 5%; left: 50%">
          <v-btn depressed elevation="2" icon style="background-color: rgba(255, 255, 255, 1)" class="btn-size3" @click="buttonClicked(true)"
            ><v-icon color="#000">mdi-plus</v-icon></v-btn
          >
          <v-btn depressed elevation="2" icon style="background-color: rgba(255, 255, 255, 1)" class="btn-size3" @click="buttonClicked(false)"
            ><v-icon color="#000">mdi-minus</v-icon></v-btn
          >
          <v-btn depressed elevation="2" icon style="background-color: rgba(255, 255, 255, 1)" class="btn-size3" @click="gridVisible = !gridVisible"
            ><v-icon color="#000">mdi-grid</v-icon></v-btn
          >
        </div>
        <div
          class="d-flex h-100"
          style="position: absolute; right: 0; transition: all 0.5s ease-in-out; align-items: flex-start; pointer-events: none"
          :style="{ right: dataBoxClose ? '0' : '-26.57143rem' }"
        >
          <dataSettingTab
            :sidebox-selected="sideboxSelected"
            :setting-modeling-list="saveMeshAniSet"
            :setting-modeling-ani-name="meshAniNameList"
            :ani-mapping-click-num="aniMappingClickNum"
            @data-box-close="getdataBoxClose"
            @now-setting-scene-ani="nowSettingSeceneAni"
            @delete-ani="deleteAni"
          ></dataSettingTab>
        </div>

        <BabylonCanvas
          ref="babylonCanvas"
          style="flex-grow: 1; transition: all 0.5s ease-in-out; height: 100%; display: flex; justify-content: center; align-items: center"
          :class="{ 'with-sidebar': dataBoxClose }"
          :grid-visible="gridVisible"
          :make-scene-ani="makeSceneAniSet"
          @reset-click-component="resetClickComponent"
          @modeling-list="getModelingList"
          @model-list-one="getMoedlingListOne"
          @delete-click="deleteClick"
        >
        </BabylonCanvas>
      </div>
    </div>
  </v-app>
</template>

<script>
import { CmsComponentMixins } from '~/plugins/mixins/cms-component'

export default {
  components: {
    DtSubNav: () => import('~/components/viewtrack/source/cms/nav/dt-sub-nav'),
    mediaList: () => import('~/components/viewtrack/source/3d-media-list.vue'),
    dataSettingTab: () => import('~/components/viewtrack/source/3d-data-setting.vue'),

    BabylonCanvas: () => import('~/components/viewtrack/source/3d-canvas.vue'),
  },
  mixins: [...CmsComponentMixins],

  layout: 'dt-builder',
  props: { selectedTpl: { type: Object, default: () => {} }, isDialog: { type: Boolean, default: false } },
  data() {
    return {
      title: '',
      tagData: [],
      allTags: [],
      sideboxSelected: 'media',
      finishDialog: false,
      gridVisible: false,

      getDownloadModeling: {}, //현재 다운받은 모델링 정보
      settingModelingList: {}, //실제 모델링 mesh 정보 전부
      nowSettingModeling: {},

      treeClickId: '',

      meshMappingList: {}, //현재 매핑 리스트
      meshMappingListLengthZero: true, //매핑한거 있는지 없는지 확인
      meshMapping: [],

      saveSceneAniSet: [], //저장할 애니 데이터
      meshAniList: [], //가공한 애니 데이터(저장)
      meshAniNameList: [], //현재 클릭 객체 애니 리스트

      makeSceneAniSet: {}, //현재 셋팅하고 있는 애니

      eventSettingList: [],
      animationClickGroup: [],
      lastSelectedIndex: null,

      showMenu: false,
      menuX: 0,
      menuY: 0,
      contextMenuItems: [{ title: '그룹화하기' }],

      componentId: '',
      currentData: {},
      loadData: {},
      data: [],
      mediaSet: [],

      saveMediaSet: [],
      nowSettingId: '',
      recentSettingId: '',
      recentMeshId: '',
      useModel: {},

      isFirstCreate: true,

      panel: [0],
      animationSelect: 'mesh',
      saveMeshAniSet: [], //메시애니 데이터
      animationMappingList: {}, //애니메이션 매핑 리스트
      dataBoxClose: true, //오른쪽 box onoff
      aniMappingClickNum: [],
    }
  },
  computed: {
    darkMode() {
      return this.$vuetify.theme.dark
    },
  },
  watch: {
    dataBoxClose(newVal) {
      this.$refs.babylonCanvas.handleResize()
    },
    sideboxSelected(newVal) {
      if (newVal === 'layer') {
        this.dataBoxClose = true
      }
    },
    meshMappingList(newVal) {
      if (Object.keys(this.meshMappingList).length === 0) this.meshMappingListLengthZero = true
      else this.meshMappingListLengthZero = false
    },
    getDownloadModeling: {
      handler: async function (newVal) {
        if (newVal?.properties) {
          this.allTags = newVal.properties.tag
        }
      },
      immediate: true,
    },
    animationMappingList(newVal) {
      this.$refs.aniMapping.setMeshAni(newVal)
    },
  },
  mounted() {
    document.addEventListener('click', this.handleDocumentClick)
    this.nowSettingId = this.generateRandomIdmodel()
  },
  beforeDestroy() {
    document.removeEventListener('click', this.handleDocumentClick)
  },
  methods: {
    async init() {
      const {
        data: { rows },
      } = await this.$axios.get('api/component')
      this.componentId = this.$route?.query._id || ''
      const matchedRow = rows.find(row => row.properties.builder === '3D' && row._id === this.componentId)
      this.loadData = matchedRow ? matchedRow : []

      this.data = matchedRow ? matchedRow.properties.data : []
      this.saveMediaSet = matchedRow ? matchedRow.properties.mediaSet : []
      this.title = matchedRow ? matchedRow.properties.name : ''
      this.tagData = matchedRow ? matchedRow.properties.tag : []
    },
    async onClickSubNavBtn(btn) {
      this.$refs.mediaListClickCom.sendMeshMapping()
      this.setUseModel()
      this.recentSettingId = this.nowSettingId
      //this.setUseModel(this.nowSettingModeling.meshId, this.nowSettingId)
      //저장 버튼
      if (btn === 'onClickSave') {
        if (this.componentId) {
          await this.updateComponent()
        } else {
          await this.createComponent()
        }
        this.$router.push({ path: '/viewtrack/p/component' })
      }
      if (btn === 'onClickSaveAs') {
        await this.createComponent()
        this.$router.push({ path: '/viewtrack/p/component' })
      }
      if (btn === 'preview') {
        //this.previewtest()
      }
      // console.info(this.saveMediaSet)
    },

    async updateComponent() {
      //다시 저장
      //this.setSaveAniDate()
      try {
        const response = await this.$axios.put('api/component', {
          _id: this.componentId,
          templateId: 'Hu5JqI8BP9d_MRtzinAl',
          properties: {
            name: this.title,
            tag: this.allTags,
            builder: '3D',
            data: this.data,
            mediaSet: this.saveMediaSet,
          },
        })
      } catch (error) {
        console.error('Error:', error.response ? error.response : error.message)
      }
    },
    async createComponent() {
      //새로 저장

      try {
        await this.$axios.post('api/component', {
          templateId: 'Hu5JqI8BP9d_MRtzinAl',
          properties: {
            name: this.title,
            tag: this.allTags,
            builder: '3D',
            data: this.data,
            mediaSet: this.saveMediaSet,
          },
        })
      } catch (error) {
        console.error('Error:', error.response ? error.response : error.message)
      }
    },
    getdataBoxClose(newVal) {
      this.dataBoxClose = newVal
    },
    getsideboxSelected(newVal) {
      this.sideboxSelected = newVal
    },
    setSaveAniDate() {
      this.saveMediaSet.forEach((item, index) => {
        item.sceneAnimation = this.saveMeshAniSet[index]
      })
    },
    getAnimationGroupName(animationKey) {
      // 안전하게 접근하기 위해 Optional Chaining 사용
      return this.nowSettingModeling?.mesh?.animationGroups?.[animationKey]?.name || ''
    },
    sendIson(isOn, child, word) {
      this.$refs.babylonCanvas.setEyes(isOn, child, word)
    },
    sendLockIson(isOn, child) {
      this.$refs.babylonCanvas.setLock(isOn, child)
    },
    handleDocumentClick(event) {
      if (!event.target.className.includes('anilist')) {
        this.aniMappingClickNum = []
      }
    },
    nowClickTemp(newVal) {
      if (this.nowSettingId != newVal) {
        this.nowSettingId = newVal
        this.setUseModel(newVal)
        this.getMeshAniList('needName')
        this.$refs.mediaListClickCom.changeMappingData(this.useModel.meshMapping)
      }
    },
    getFilteredItems(item) {
      const newitem = { ...item } // 객체 복사
      const entries = Object.entries(newitem) // 복사된 객체의 키-값 쌍을 배열로 변환
      for (let i = entries.length - 1; i >= 0; i--) {
        const [key, value] = entries[i]
        if (!key.startsWith('ani') && !key.startsWith('group')) {
          this.$delete(newitem, key)
        }
      }
      return newitem
    },
    handleRowClick(item, key, event) {
      if (event.shiftKey && this.lastSelectedIndex !== null) {
        // 이미 키가 있는지 확인하여 없으면 추가, 있으면 제거
        const index = this.aniMappingClickNum.indexOf(key)
        if (index === -1) {
          this.aniMappingClickNum.push(key)
        } else {
          this.aniMappingClickNum.splice(index, 1)
        }

        const currentIndex = key
        const [start, end] = [this.lastSelectedIndex, currentIndex].sort((a, b) => a - b)

        for (let i = start; i <= end; i++) {
          if (!this.animationClickGroup.includes(i)) {
            this.animationClickGroup.push(i)
          } else {
            const groupIndex = this.animationClickGroup.indexOf(i)
            this.animationClickGroup.splice(groupIndex, 1)
          }
        }
      } else {
        if (this.animationClickGroup.includes(key)) {
          this.animationClickGroup = this.animationClickGroup.filter(k => k !== key)
        } else {
          this.animationClickGroup.push(key)
        }
        this.lastSelectedIndex = key
        this.aniMappingClickNum = []
        this.aniMappingClickNum.push(key)
      }
    },
    resettingModelingList(newVal) {
      this.saveMediaSet = this.saveMediaSet.filter(item => item.mediaTemplateId !== newVal)

      this.$refs.babylonCanvas.clearAllGlb(newVal)
    },
    handleAddTag(tags) {
      const uniqueTags = [...new Set(tags)]
      this.allTags = uniqueTags
    },
    showContextMenu(event, key) {
      event.preventDefault()

      if (this.aniMappingClickNum.length > 1) {
        this.menuX = event.clientX
        this.menuY = event.clientY
        this.showMenu = true
      }
    },
    handleMenuItemClick(menuItem) {
      //그룹만들기
      const groupKey = `group_${Date.now()}`
      const newGroup = {}
      this.aniMappingClickNum.forEach(key => {
        newGroup[key] = this.animationMappingList[key]
        delete this.animationMappingList[key]
      })
      newGroup.name = '그룹'
      this.$set(this.animationMappingList, groupKey, newGroup)
      this.aniMappingClickNum = []
      this.showMenu = false
    },
    clickDeviceId(newVal) {
      //id매핑 리스트 클릭시 메쉬, 장치 연결
      if (this.treeClickId) {
        //레이어 클릭 했는지 확인
        const isIN = Object.values(this.meshMappingList).some(item => item.meshid === this.treeClickId) //이미 매핑 됐는지 확인
        if (isIN) {
          //이미 매핑됐을 경우
          for (const key in this.meshMappingList) {
            if (this.meshMappingList[key].meshid === this.treeClickId) {
              this.$set(this.meshMappingList[key], 'deviceid', newVal)
            }
          }
        } else {
          //새로 매핑 추가
          if (this.treeClickId) {
            const generateId = this.generateRandomId()
            this.$set(this.meshMappingList, generateId, {
              meshid: this.treeClickId,
              deviceid: newVal,
            }) //메쉬id, 장치id
          }
        }
      }
    },
    deleteMapplingClick(getkey, list) {
      //매핑 삭제
      for (const key in list) {
        if (key === getkey) {
          this.$delete(list, key)
        } else if (typeof list[key] === 'object' && list[key] !== null) {
          this.deleteMapplingClick(getkey, list[key])
        }
      }
    },
    generateRandomId() {
      return 'id-' + Math.random().toString(36).substr(2, 9) + '-' + Date.now()
    },
    generateRandomIdSave() {
      return 'save-' + Math.random().toString(36).substr(2, 9) + '-' + Date.now()
    },
    deleteClick() {
      //선택 레이어 아이디 초기화
      //this.$refs.mediaListClickCom.resetColor()
    },
    truncateFilename(name) {
      const maxLength = 10
      if (name) {
        if (name.length > maxLength) {
          return name.substring(0, maxLength) + '...'
        } else {
          return name
        }
      }
      return
    },
    nowSettingSeceneAni(newVal) {
      this.makeSceneAniSet = newVal
    },
    getModelingList(newVal) {
      this.settingModelingList = newVal
    },
    getMoedlingListOne(newVal) {
      this.nowSettingModeling = newVal
      if (this.useModel.meshAnimation === undefined || this.useModel.meshAnimation?.length === 0) {
        this.getMeshAniList('', this.saveMeshAniSet)
      } else {
        this.getMeshAniList('needName')
      }
      this.saveMeshAniSet = this.meshAniList
    },
    inputTitle(title) {
      this.title = title
    },
    resetClickComponent() {
      //객체 삭제 초기화
      this.getDownloadModeling = {}
      this.$refs.mediaListClickCom.resetComponent()
    },
    buttonClicked(gettype) {
      this.$refs.babylonCanvas.handleParentClick(gettype)
    },
    dataTemplateBoxBtn() {
      this.dataBoxClose = !this.dataBoxClose
    },
    nowSettingMapping(newVal) {
      this.meshMapping = newVal
    },
    async clickComponent(newVal, media = '', resolve) {
      if (media === '') {
        //처음생성
        this.setUseModel()
        this.nowSettingId = this.generateRandomIdmodel()
        newVal.createId = this.nowSettingId
        await this.$refs.babylonCanvas.loadGlb(newVal)

        this.useModel.mediaTemplateId = this.nowSettingId
        this.useModel.mediaId = newVal._id
        this.useModel.meshAnimation = []
        this.getDownloadModeling = newVal
      } else {
        //이전 기록 불러오기
        this.useModel = media
        this.nowSettingId = media.mediaTemplateId
        newVal.createId = this.nowSettingId
        await this.$refs.babylonCanvas.loadGlb(newVal)
        this.getDownloadModeling = newVal
      }
      if (resolve) {
        resolve()
      }
    },
    setUseModel(childCreateId = '') {
      //객체에 대한 데이터 모델 생성 getId: mediaId
      if (this.useModel.mediaId != null) {
        //처음 생성이 아닐 때 기존것을 저장. 똑같은 객체 중복 생성 가능
        this.useModel.meshMapping = this.meshMapping
        this.useModel.meshAnimation = this.saveMeshAniSet
        let index = this.saveMediaSet.findIndex(item => item.mediaTemplateId === this.useModel.mediaTemplateId)
        if (index !== -1) {
          this.$set(this.saveMediaSet, index, {
            ...this.saveMediaSet[index],
            ...this.useModel,
          })
        } else {
          this.saveMediaSet.push({
            ...this.useModel,
          })
        }
        this.useModel = {}
        if (childCreateId != '') {
          //현재 사용할 객체의 mappinglist 기록 불러오기
          // console.info(this.settingModelingList)
          // this.nowSettingModeling = this.settingModelingList[this.nowSettingId]

          for (let i = 0; i < this.saveMediaSet.length; i++) {
            if (this.saveMediaSet[i].mediaTemplateId === childCreateId) {
              this.useModel = this.saveMediaSet[i]
              this.meshAniList = this.useModel.meshAnimation
              this.saveMeshAniSet = this.meshAniList
              break
            }
          }
        } else {
          this.meshMappingList = {}
        }
        this.recentSettingId = this.nowSettingId
      } else {
        this.meshMappingList = {}
        if (this.saveMediaSet.length != 0) {
          this.useModel = this.saveMediaSet[0]
          let index = this.saveMediaSet.findIndex(item => item.mediaTemplateId === this.useModel.mediaTemplateId)
          this.useModel.meshAnimation = this.saveMediaSet[0].meshAnimation
          this.useModel.meshMapping = this.saveMediaSet[0].meshMapping
        }
      }
    },
    generateRandomIdmodel() {
      const randomString = Math.random().toString(36).substr(2, 9) // 9자리 랜덤 문자열 생성
      return `model${randomString}`
    },
    deleteAni(newVal) {
      this.$refs.babylonCanvas.deleteAni(newVal)
    },
    getMeshAniList(getNeed = '', anilist = []) {
      this.meshAniNameList = []
      this.meshAniList = []
      if (this.nowSettingModeling != null) {
        let foundItem = null

        for (const key in this.settingModelingList) {
          if (this.settingModelingList[key].createId === this.nowSettingId) {
            foundItem = this.settingModelingList[key]
            break
          }
        }

        // this.nowSettingModeling = foundItem
        if (foundItem != null) {
          let originalAni = foundItem.mesh?.animationGroups
          if (originalAni != null) {
            if (getNeed === 'needName') {
              originalAni?.forEach((item, index) => {
                this.meshAniNameList.push(item.name)
              })
              this.meshAniList = this.useModel.meshAnimation
            } else {
              originalAni?.forEach((item, index) => {
                // 현재는 이름처리x
                this.meshAniList.push({ animationIndex: [index], disable: [true] })
                this.meshAniNameList.push(item.name)
              })
            }
          }
        }
      }
      //this.$emit('make-ani', this.meshAniList)
    },
  },
}
</script>

<style>
.builder-class p {
  margin: 0 !important;
}
.builder-class ::-webkit-scrollbar {
  width: 0.4rem;
}
.builder-class ::-webkit-scrollbar-thumb {
  background-color: #d5d5d5;
  border-radius: 10px;
}
.builder-class ::-webkit-scrollbar-track {
  background: #fff;
  border-radius: 10px;
}
.builder-class .v-input__slot:before {
  border-color: rgba(0, 0, 0, 0) !important;
}
.builder-class .v-btn-toggle:not(.v-btn-toggle--dense) .v-btn.v-btn.v-size--default {
  height: fit-content;
}
.builder-class .v-list-item {
  padding: 0;
}
.builder-class .custom-checkbox .v-icon {
  font-size: 16px;
  height: 16px;
  width: 16px;
}
.dialog-checkbox-div .v-input--selection-controls__input {
  align-items: center;
  width: fit-content;
}
.dialog-checkbox-div i {
  width: 1rem !important;
  height: 1rem !important;
  font-size: 1rem !important;
}
.dialog-checkbox .v-input--selection-controls__ripple {
  left: -17px;
}

.side-box-select-checkbtn .v-btn--hover {
  background-color: rgba(126, 115, 254, 0.15) !important;
  border-radius: 0.21429rem !important;
  color: #ffffff !important;
  /* 보라색 */
}
.side-box-select-checkbtn .v-btn--active {
  background-color: rgba(126, 115, 254, 0.15) !important;
  border-radius: 0.21429rem !important;
  color: #ffffff !important;
  /* 보라색 */
}
.btn-size3 {
  min-width: 3rem;
  min-height: 3rem;
}

.finishDialog-main {
  box-shadow: 0px 3px 7px 0px rgba(0, 0, 0, 0.12);
  border-radius: 0.21429rem;
  padding-top: 1.93rem;
}
.dialog-set-main .v-messages {
  min-height: 0;
}
.dialog-set-main .v-input--selection-controls {
  margin-top: 0;
}
.dialog-set-main .v-label {
  font-size: 1rem;
}
.dialog-set-main .v-size--default {
  min-width: 0 !important;
}
.dialog-select-btn {
  padding: 0.86rem 0rem !important;
  width: 4.42857rem;
}
.search-text-field {
  background-color: #f9fafb;
  padding: 0 !important;
  margin: 0 !important;
}
.search-text-field input {
  background-color: #f9fafb;
}
.builder-side-box {
  width: 20.28571rem;
  /* margin: 0.75rem 0rem; 박스 위아래 붙임*/
  border-radius: 0.35714rem;
  /* border-left: 2px solid #f8f8f8; */
  background-color: #fff;
  box-shadow: 2px 2px 5px 0px rgba(0, 0, 0, 0.05);
}
.v-app-class .v-application--wrap {
  min-height: 0% !important;
  display: flex;
  flex-direction: column;
}
.v-app-class .v-stepper .v-stepper__header .v-stepper__step {
  flex-basis: auto;
  min-width: 100px;
}

.v-app-class .v-stepper .v-divider {
  max-width: 3.5rem;
}
.data-template-box-right-btn {
  padding: 1.5rem 0rem 1.5rem 0rem;
  background-color: #fff;
  height: fit-content;
  box-shadow: 0px 0px 7px 0px rgba(0, 0, 0, 0.12);
  border-radius: 0.21429rem 0rem 0rem 0.21429rem;
}
.data-template-box-right-btn .v-btn--active {
  box-shadow: none !important;
  background-color: transparent !important;
}
.stepper-group .v-stepper__content {
  height: 100%;
}
.stepper-step:hover {
  background-color: transparent !important;
}
.stepper-step .v-stepper__step__step {
  background-color: #f8f8f9 !important;
}
.stepper-step .v-stepper__label {
  color: #7e7e7e;
}
.stepper-step:hover .v-stepper__label,
.stepper-step:hover .v-stepper__step__step {
  color: var(--g-dt-primary) !important;
  font-weight: 500;
}
.stepper-group .v-stepper__step--active .v-stepper__label,
.stepper-group .v-stepper__step--active .v-stepper__step__step {
  color: var(--g-dt-primary) !important;
  font-weight: 500;
}
.stepper-group .v-stepper__step--active .v-stepper__step__step {
  background-color: rgba(126, 115, 254, 0.16) !important;
}
.animation-btn-small {
  flex: 1;
  text-align: center;
  padding-top: 0.36rem !important;
  padding-bottom: 0.36rem !important;
  border-radius: 3.57143rem !important;
  background-color: #fff !important;
  border: none !important;
  font-size: 0.85714rem !important;
}
.animationlist-content .v-expansion-panel-content__wrap {
  display: flex;
  flex-direction: column;
}
.no-select {
  /* 마우스 글씨 스크롤 방지 */
  -webkit-user-select: none; /* Safari */
  -moz-user-select: none; /* Firefox */
  -ms-user-select: none; /* Internet Explorer/Edge */
  user-select: none; /* Non-prefixed version, currently supported by Chrome, Opera and Edge */
}
.aniList-item .v-list-group__header {
  height: 2.71419rem;
  align-items: center;
  padding: 0 !important;
  min-height: 0;
}
.aniList-item .v-list-item__icon {
  align-self: normal !important;
}
.aniList-item .v-list-item {
  height: 2.71419rem;
  padding: 0 0 0 0.57rem !important;
  min-height: 0;
}
.aniList-item .v-list-group .v-list-group__items .v-list-item--link {
  padding: 0;
  padding-left: 2.71419rem !important;
}
.with-sidebar {
  margin-right: 26.57143rem;
}
.builder-class .v-expansion-panel-content__wrap {
  padding-left: 1.14rem !important;
  padding-right: 1.14rem !important;
}
</style>
