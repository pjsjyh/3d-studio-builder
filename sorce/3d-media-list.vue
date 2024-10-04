<template>
  <section class="f-row w-100 f-full" style="flex: 1">
    <v-btn-toggle
      v-model="sideboxSelected"
      class="side-box-select-checkbtn flex-column d-flex gap-4"
      mandatory
      style="background-color: var(--DT-light-dt_bg_2, #fcfdfe) !important; padding: 1.29rem 1.07rem"
    >
      <v-btn class="side-box-select-checkbtn-btn border-0" value="media" large icon>
        <i class="dti-media" style="font-size: 2.6rem" :style="{ color: 'var(--g-dt-primary)' }" />
      </v-btn>
      <v-btn class="side-box-select-checkbtn-btn border-0" value="layer" large icon>
        <i class="dti-layer" style="font-size: 2.6rem" :style="{ color: 'var(--g-dt-primary)' }" />
      </v-btn>
    </v-btn-toggle>
    <div
      class="builder-side-box d-flex"
      style="width: 21.42857rem"
      :style="{ background: darkMode ? '#1F1F1F' : '#fff', borderLeft: '2px solid var(--DT-light-dt_bg_3, #F8F8F9)' }"
    >
      <div class="builder-side-box-midea w-100 h-100 d-flex flex-column">
        <template v-if="!clickSearch">
          <div style="padding: 0.57rem 2rem">
            <div class="d-flex justify-content-between align-items-center h-6">
              <div class="builder-side-box-title fw-bold" style="font-size: 1.14286rem; color: var(--DT-light-secondary, #4136c0)">
                {{ sideboxSelected === 'layer' ? '레이어' : 'DT 파일' }}
              </div>
              <div class="d-flex">
                <v-btn class="btn-size3" icon :style="{ display: sideboxSelected === 'layer' ? 'none' : 'flex' }" @click="mediaDownloadDialog = true"
                  ><v-icon class="dti-download"></v-icon
                ></v-btn>
                <v-btn
                  class="btn-size3"
                  icon
                  @click="
                    {
                      searchToOn()
                    }
                  "
                  ><v-icon class="dti-search"></v-icon
                ></v-btn>
              </div>
            </div>
          </div>
        </template>
        <template v-else-if="clickSearch">
          <div class="builder-side-box-search d-flex flex-column py-2 px-3" :style="{ height: !inputSearchCheck ? '100%' : 'fit-content' }">
            <div class="d-flex align-items-center builder-side-box-search-title h-5 mb-2">
              <v-btn
                icon
                style="width: fit-content; margin-right: 0.95rem"
                @click="
                  {
                    clickSearch = false
                  }
                "
                ><v-icon>mdi-chevron-left</v-icon></v-btn
              >
              <div style="font-size: 1.14286rem; color: rgba(85, 74, 211, 1); font-weight: 600">검색</div>
            </div>
            <div class="w-100 d-flex align-items-center" style="height: 3rem; background-color: #f9fafb; padding: 0.44rem 0.88rem">
              <v-icon class="builder-side-box-search-icon mr-1 dti-search" style="width: 1.75rem; height: 1.75rem" small></v-icon>
              <v-text-field v-model="inputSearchValue" class="w-100 h-100 search-text-field" label="검색"></v-text-field>
            </div>
            <MediaListEmpty :get-word="`검색해 보세요`" :get-img="'dti-search'" :is-visible="inputSearchCheck"></MediaListEmpty></div
        ></template>
        <div v-show="sideboxSelected == 'media'">
          <DtMedia
            :click-search="clickSearch"
            :download-modeling-list="downloadModelingList"
            :input-search-check="inputSearchCheck"
            @click-component="clickComponent"
          ></DtMedia>
        </div>
        <div
          v-show="sideboxSelected == 'layer'"
          style="padding-right: 2rem; flex: 1; display: flex; flex-direction: column; padding-left: 2rem; padding-bottom: 1rem"
        >
          <v-checkbox
            v-model="showMappingList"
            dense
            style="margin-top: 0 !important; padding-left: 0.8rem; font-size: 1rem"
            :label="`매핑한 레이어`"
          ></v-checkbox>
          <div
            class="layer_div"
            style="
              height: 39.92857rem;
              flex: 1;
              overflow-y: auto;
              overflow-x: hidden;
              padding-right: 0.5rem;
              display: flex;
              flex-direction: column;
              padding-bottom: 1rem;
            "
          >
            <div :style="{ display: !modalListCheck ? 'none !important' : !clickSearch ? 'block' : inputSearchCheck ? 'block' : 'none' }">
              <div ref="layerList" class="tree-node">
                <v-list-group
                  v-for="(rootNode, index) in setClickObj"
                  :key="rootNode.id"
                  :value="true"
                  no-action
                  sub-group
                  class="d-flex flex-column root-node"
                  color="black"
                  @click="childTreeClick(rootNode)"
                  @contextmenu.prevent="showContextMenu($event, rootNode.id)"
                >
                  <template #activator>
                    <v-list-item-content class="d-flex justify-content-between">
                      <v-list-item-title>{{ rootNode.name }}</v-list-item-title>
                    </v-list-item-content>
                    <div class="d-flex align-items-center">
                      <IconOnOff
                        class="d-flex aligh-items-center"
                        style="width: 2rem; height: 2rem"
                        :first-icon="'dti-eye-open'"
                        :second-icon="'dti-eye-close'"
                        :get-size="2"
                        @send-ison="value => sendIson(value, rootNode)"
                      ></IconOnOff>
                      <IconOnOff
                        class="d-flex aligh-items-center"
                        style="width: 2rem; height: 2rem"
                        :first-icon="'dti-unlock'"
                        :get-size="2"
                        :second-icon="'dti-lock'"
                        @send-ison="value => sendLockIson(value, rootNode)"
                      ></IconOnOff>
                      <v-btn style="width: 2rem; height: 2rem" depressed icon @click="handleMenuItemClick(rootNode.id, true)"
                        ><v-icon>dti-trashcan</v-icon></v-btn
                      >
                    </div>
                  </template>
                  <div style="margin-bottom: 0.36rem"></div>
                  <v-list-item
                    v-for="child in rootNode.children"
                    :key="child.id"
                    link
                    class="child-node g-cursor-pointer d-flex align-items-center justify-content-between"
                    style="height: 2.4rem"
                    :style="{
                      backgroundColor: getBackgroundColor(child.name),
                      display: showMappingList ? (child.isMapping ? 'block' : 'none !important') : 'block',
                    }"
                    @click="childTreeClick(child)"
                    @mouseover="hoverChildId = child.name"
                    @mouseleave="hoverChildId = null"
                  >
                    <v-list-item-title>{{ child.name }}</v-list-item-title>
                    <div class="d-flex align-items-center">
                      <IconOnOff
                        :first-icon="'dti-eye-open'"
                        :second-icon="'dti-eye-close'"
                        :get-size="2"
                        @send-ison="value => sendIson(value, child)"
                      ></IconOnOff>
                      <IconOnOff
                        :first-icon="'dti-unlock'"
                        :second-icon="'dti-lock'"
                        :get-size="2"
                        @send-ison="value => sendLockIson(value, child)"
                      ></IconOnOff>
                      <IconOnOff
                        :first-icon="'dti-unlink'"
                        :second-icon="'dti-link'"
                        :get-size="2"
                        :wait-version="true"
                        :is-mapping="child.isMapping"
                        @send-ison="isOn => sendLinkIson(isOn, child)"
                      ></IconOnOff>
                    </div>
                  </v-list-item>
                </v-list-group>
                <v-menu v-model="showMenu" :position-x="menuX" :position-y="menuY" absolute offset-y>
                  <v-list>
                    <v-list-item v-for="(menuItem, index) in contextMenuItems" :key="index" @click="handleMenuItemClick(menuItem)">
                      <v-list-item-title>{{ menuItem.title }}</v-list-item-title>
                    </v-list-item>
                  </v-list>
                </v-menu>
              </div>
            </div>
            <MediaListEmpty
              :style="{ display: modalListCheck ? 'none !important' : 'block' }"
              :get-word="`레이어가 없습니다`"
              :get-img="`dti-layer`"
              :img-size="'3.588rem'"
              :is-visible="!inputSearchCheck"
            ></MediaListEmpty>
          </div>
        </div>
      </div>
    </div>
    <div
      class="position-absolute bg-white"
      style="right: -110%; top: 1.5rem; padding: 0.64rem 2rem 1.71rem 2rem; border-radius: 0.71429rem; box-shadow: 0px 3px 7px 0px rgba(0, 0, 0, 0.1)"
      :style="{ display: isMappingOn ? 'block' : 'none' }"
    >
      <v-btn
        class="w-5 h-5 pr-3"
        style="position: absolute; right: 0"
        depressed
        icon
        @click="
          isMappingOn = false
          tab = 0
          mappingClickId = ''
        "
        ><v-icon>mdi-close</v-icon></v-btn
      >
      <v-tabs v-model="tab" style="width: 23.714rem; padding-top: 2rem; padding-bottom: 1.43rem" color="var(--DT-light-dt_secondary, #4136C0)">
        <v-tab style="flex: 1" value="1">매핑 리스트</v-tab>
        <v-tab style="flex: 1" value="2">POI</v-tab>
      </v-tabs>
      <v-tabs-items v-model="tab" style="">
        <v-tab-item>
          <v-card flat class="d-flex flex-column align-items-center justify-content-center">
            <IdMapping @click-device-id="clickDeviceId"></IdMapping>
          </v-card>
        </v-tab-item>
        <v-tab-item>
          <v-card flat class="d-flex flex-column">
            <PoiMapping></PoiMapping>
          </v-card>
        </v-tab-item>
      </v-tabs-items>
      <div class="d-grid w-100" style="gap: 0.71rem; grid-template-columns: auto auto; margin-top: 1.43rem">
        <v-btn depressed class="" @click="mappingReset">초기화</v-btn>

        <v-btn depressed color="var(--DT-light-dt_primary_active, rgba(126, 115, 254, 0.16))" @click="handleClick"> {{ tab === 0 ? '다음' : '추가' }}</v-btn>
      </div>
    </div>
    <v-dialog v-model="mediaDownloadDialog" class="media-dialog" style="width: 50% !important; height: 55rem" max-width="1102">
      <div class="media-download-modal d-flex bg-white position-relative">
        <div class="media-download-modal-left">
          <div class="media-download-modal-img w-100 d-flex position-relative" style="gap: 1rem">
            <div class="media-modal-img-main w-100 d-flex justify-content-center bg-black align-items-center">
              <template v-if="selectedItem.properties.thumbnailPath">
                <model-viewer
                  :key="selectedItem.properties.path"
                  reveal="manual"
                  :src="`${gractorBaseUrl}/api/media/download?path=${selectedItem.properties.path}`"
                  style="display: block; width: 100%; height: 100%"
                  camera-controls
                >
                  <v-img
                    id="lazy-load-poster"
                    ref="modelViewerPoster"
                    slot="poster"
                    class="position-absolute media-modal-img-main-img"
                    :style="{ backgroundImage: 'url(' + `${gractorBaseUrl}/api/media/download?path=${selectedItem.properties.thumbnailPath}` + ')' }"
                  ></v-img
                ></model-viewer>
              </template>
              <template v-else>
                <model-viewer
                  :key="selectedItem.properties.path"
                  :src="`${gractorBaseUrl}/api/media/download?path=${selectedItem.properties.path}`"
                  style="display: block; width: 100%; height: 100%"
                  camera-controls
                ></model-viewer>
              </template>
            </div>
          </div>
          <div class="media-download-modal-left-title justify-content-between position-relative">
            <div class="modal-title-group">
              <p class="m-0">3D</p>
              <p class="m-0 font-weight-bold" style="font-size: 1.42857rem; color: #171717; font-weight: 600; margin-top: 0.86rem">
                {{ truncateFilename(selectedItem.properties.name) }}
              </p>
            </div>
            <div class="media-download-modal-tag d-flex">
              <v-chip v-for="(option, index) in selectedItem.properties.tag" :key="index" class="chip-set tag-chip">
                {{ option }}
              </v-chip>
            </div>
            <v-btn class="modal-title-download" depressed :disabled="selectedItem.downloaded" @click="downloadModeling(selectedItem)"
              ><v-icon class="btn-size3 dti-download" color="rgba(126, 115, 254, 1)"></v-icon>다운로드</v-btn
            >
          </div>

          <div class="media-download-modal-info d-flex flex-column" style="gap: 0.5rem; margin-top: 2.64rem; margin-bottom: 5.07rem">
            <div class="d-flex">
              <p class="m-0 media-modal-info-title">파일 크기</p>
              <p class="m-0 media-modal-info-detail">{{ formattedSize }}</p>
            </div>
            <div class="d-flex">
              <p class="m-0 media-modal-info-title">업로드 날짜</p>
              <p class="m-0 media-modal-info-detail">{{ formattedUploadDate }}</p>
            </div>
          </div>
        </div>
        <div class="media-download-modal-line h-90 position-absolute" style="margin-left: 40.43rem"></div>
        <div class="media-download-modal-right d-flex flex-column h-100" style="flex: 1">
          <div class="media-download-modal-right-top d-flex justify-content-end">
            <v-btn class="media-download-modal-close" icon @click="mediaDownloadDialog = false"><v-icon>mdi-close</v-icon></v-btn>
          </div>
          <span style="font-size: 1.14286rem; font-weight: 600; padding: 0.71rem; margin-bottom: 0.64rem">파일 다운로드</span>
          <div style="padding-right: 2rem">
            <div style="width: 95%; margin-bottom: 1.14rem">
              <Selectbox3d
                id="selectboxDownloadModal"
                :is-filter-icon="true"
                :solution-on="true"
                :type-on="false"
                :tag-on="true"
                :use-tag-list="true"
                :get-list="modalList"
                :get-tag-list="modelingList"
                @selected-option="getModalSelectOption"
              ></Selectbox3d>
            </div>
            <div class="media-download-model-list d-flex position-relative mr-1" style="display: block !important">
              <v-list class="w-100" style="gap: 0.44rem; display: grid; padding-right: 0.5rem">
                <v-list-item
                  v-for="item in modelingList"
                  :key="item._id"
                  class="modalview-layer w-100 d-flex align-items-center g-cursor-pointer position-relative mr-3 p-3 mb-3"
                  :class="{ 'selected-item': selectedItem._id === item._id }"
                  :style="{ display: checkModalTagIn(item, getModalSelectedOption) ? 'flex !important' : 'none !important' }"
                  style="justify-content: space-between; border-radius: 0.35714rem; border: 1px solid var(--DT-light-dt_bg_3, #f8f8f9)"
                  @click="selectItem(item)"
                >
                  <div class="d-flex" style="flex-grow: 1">
                    <div class="d-flex justify-content-center media-download-model-list-img" style="flex: 1">
                      <template v-if="item.properties.thumbnailPath">
                        <model-viewer
                          :key="item.properties.path"
                          reveal="manual"
                          :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.path}`"
                          style="display: block; width: 100%; height: 100%"
                          camera-controls
                        >
                          <v-img
                            id="lazy-load-poster"
                            ref="modelViewerPoster"
                            slot="poster"
                            class="position-absolute media-modal-img-main-img"
                            :style="{
                              backgroundImage: 'url(' + `${gractorBaseUrl}/api/media/download?path=${item.properties.thumbnailPath}` + ')',
                              maxWidth: '100%',
                              maxHeight: '100%',
                            }"
                          ></v-img
                        ></model-viewer>
                      </template>
                      <template v-else>
                        <model-viewer
                          :key="item.properties.path"
                          :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.path}`"
                          style="display: block; width: 100%; height: 100%"
                          :style="{ maxWidth: '100%', maxHeight: '100%' }"
                          camera-controls
                        ></model-viewer>
                      </template>
                      <!-- <img
                        class="img-fluid h-100"
                        :src="`${gractorBaseUrl}/api/media/download?path=${item.properties.thumbnailPath}`"
                        :style="{ maxWidth: '100%', maxHeight: '100%' }"
                      /> -->
                    </div>
                    <div style="gap: 0.69rem; display: grid; align-items: center; position: relative; flex: 1">
                      <p class="m-0" style="font-size: 1rem">
                        {{ item.properties.name === '' ? '   ' : truncateFilename(item.properties.name) }}
                      </p>
                      <!-- <div style="display: flex; gap: 0.44rem">
                      <v-chip v-for="(option, index) in item.tag" :key="index" class="chip-set tag-chip">
                        {{ option }}
                      </v-chip>
                    </div> -->
                      <div style="width: 100%">
                        <TagList :select-tag-array="item.properties.tag"></TagList>
                      </div>
                    </div>
                  </div>

                  <v-btn style="background-color: #fff; filter: drop-shadow(0px 2px 3px rgba(0, 0, 0, 0.05))" icon @click="downloadModeling(item)"
                    ><v-icon class="btn-size3" :class="!item.downloaded ? 'dti-download' : ''" :color="item.downloaded ? '#7E73FE' : '#000000'">{{
                      item.downloaded ? 'mdi-check' : ''
                    }}</v-icon>
                  </v-btn>
                </v-list-item>
              </v-list>
            </div>
          </div>
        </div>
      </div>
    </v-dialog>
  </section>
</template>

<script>
import TypecheckMixins from '~/components/viewtrack/source/divide-types-mixin'
import { read as xlsxRead } from 'xlsx'
export default {
  components: {
    Selectbox3d: () => import('~/components/viewtrack/source/3d-select-box.vue'),
    TagList: () => import('~/components/viewtrack/source/3d-taglist.vue'),
    DtMedia: () => import('~/components/viewtrack/source/3d-dt-media.vue'),
    MediaListEmpty: () => import('~/components/viewtrack/source/media-list-empty.vue'),
    IconOnOff: () => import('~/components/viewtrack/source/3d-icon-onoff.vue'),
    IdMapping: () => import('~/components/viewtrack/source/3d-id-mapping.vue'),
    PoiMapping: () => import('~/components/viewtrack/source/3d-poi-mapping.vue'),
  },
  mixins: [TypecheckMixins],

  props: {
    settingModelingList: { type: Object, default: () => {} },
    mediaSet: { type: Array, default: () => [] },
  },
  data() {
    return {
      tab: 0,
      isMappingOn: false,
      nowMappingChild: '',
      sideboxSelected: 'media',
      showMappingList: false,
      modelingList: [], //다운로드 받은 모델링 list
      firstmodelingList: [],
      //모델링에 필요한 정보. 배경 이미지, 이름, 모델링크, downloaded정보, 태그
      mediaDownloadDialog: false, //모델링 다운 dialog
      selectedItem: {
        createdAt: '',
        dovType: '',
        properties: {
          filename: '',
          mimetype: '',
          name: '',
          originalname: '',
          path: '',
          size: '',
          tag: '',
          thumbnailPath: '',
          uploadDate: '',
        },
      }, //선택 item 속성
      downloadModelingList: [], //다운로드한 모델링 리스트
      modalListCheck: false,
      clickSearch: false,
      getModalSelectedOption: {},
      inputSearchValue: '',
      inputSearchCheck: false,
      modalList: [{ label: '전체보기', value: 'all' }],

      nowClickListId: '',
      clickDownList: {},
      setClickObj: [],
      selectedChildId: null, // 현재 선택된 항목의 ID를 저장
      hoverChildId: null,
      layerListParent: null,

      showMenu: false,
      menuX: 0,
      menuY: 0,
      contextMenuItems: [{ title: '삭제' }],
      admins: [
        ['Management', 'mdi-account-multiple-outline'],
        ['Settings', 'mdi-cog-outline'],
      ],
      isOn: true,

      treeClickId: '',
      mappingClickId: '',
      meshMappingList: [], //현재 매핑 리스트
      nowMappingClickTemp: '',

      getMappingData: [],
    }
  },
  computed: {
    darkMode() {
      return this.$vuetify.theme.dark
    },
    formattedUploadDate() {
      if (!this.selectedItem.createdAt) return ''
      const date = new Date(this.selectedItem.createdAt)
      return date.toLocaleString('ko-KR', { timeZone: 'Asia/Seoul' })
    },
    formattedSize() {
      if (!this.selectedItem.properties.size) return ''
      const sizeInMB = this.selectedItem.properties.size / (1024 * 1024)
      return `${sizeInMB.toFixed(2)} MB`
    },
    gractorBaseUrl() {
      return this.$config.viewtrack.gractorBaseUrl
    },
    productBaseUrl() {
      return this.$config.viewtrack.productBaseUrl
    },
  },
  watch: {
    nowMappingClickTemp(newVal) {
      // console.info(newVal)
    },
    nowClickListId(newVal) {
      // console.info(newVal)
    },
    meshMappingList() {},
    nowMappingChild(newVal) {},
    clickDownList(newVal) {
      this.nowMappingClickTemp = newVal.createId
      const newRootNodes = {
        id: newVal.createId,
        name: newVal.name,
        isMapping: this.meshMappingList.some(item => item.meshId === item.id) ? true : false,
        createId: newVal.createId,
        meshId: newVal.meshId,
        children: Object.values(newVal.mesh.meshes)
          .filter(item => item.id !== '__root__')
          .map(item => ({
            id: item.id,
            name: item.name,
            isRoot: false,
            meshId: newVal.meshId,
            createId: newVal.createId,
            isMapping: this.meshMappingList.some(mappingData => mappingData.meshid === item.id) ? true : false,
          }))
          .filter((item, index, self) => item.name && index === self.findIndex(t => t.name && t.name.includes(this.removePrimitiveSuffix(item.name)))),
      }
      this.nowMappingChild = newRootNodes
      this.setClickObj = [...this.setClickObj, newRootNodes]
    },
    settingModelingList(newVal) {
      if (newVal != this.clickDownList) this.clickDownList = newVal
    },
    sideboxSelected(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.clickSearch = false
      }
      if (newVal === 'layer') {
        this.layerListParent = this.$refs.layerList
      }
      this.$emit('sidebox-selected', this.sideboxSelected)
    },
    clickSearch(newValue) {
      if (!newValue) {
        this.inputSearchValue = ''
      }
    },
    inputSearchValue(newValue) {
      if (this.sideboxSelected === 'layer') {
        if (this.layerListParent) {
          const divs = this.layerListParent.querySelectorAll('div')
          divs.forEach(div => {
            if (div.outerText.includes(newValue)) {
              div.style.display = 'block'
            } else {
              div.style.setProperty('display', 'none', 'important')
            }
          })
          if (newValue == '') {
            this.inputSearchCheck = false
          } else {
            this.inputSearchCheck = true
          }
        }
      } else {
        if (newValue == '') {
          this.inputSearchCheck = false
          this.downloadModelingList = this.downloadModelingList.map(item => ({ ...item, onoff: true }))
        } else {
          this.inputSearchCheck = true
          this.downloadModelingList = this.downloadModelingList.map(item => ({
            ...item,
            onoff: item.properties.name.includes(newValue),
          }))
        }
      }
    },
    selectedItem(newVal) {
      //console.info(newVal)
    },
  },
  async mounted() {
    document.addEventListener('click', () => {
      this.showMenu = false
    })
    try {
      const { data: { rows: gractorMediaList = [] } = {} } = await this.$axios.get(`${this.gractorBaseUrl}/api/media`)
      if (gractorMediaList) {
        this.firstmodelingList = gractorMediaList
        const modelData = this.firstmodelingList.filter(item => this.checkType(item.properties.mimetype) === 'model')

        const promises = modelData.map(async item => {
          //item.properties.tag = this.parseTags(item.properties.tag)
          const modelBase64 = await this.fetchImageAsBase64(item.properties.filename, item.properties.path)

          return {
            ...item,
            srcPath: modelBase64,
          }
        })
        this.modelingList = await Promise.all(promises)
        if (this.mediaSet.length > 0 && this.mediaSet.mediaId != '') {
          for (const media of this.mediaSet) {
            const item = this.modelingList.find(modellist => modellist._id === media.mediaId)
            if (item) {
              // item이 존재할 때 실행할 함수
              this.meshMappingList = media.meshMapping

              this.selectedItem = item
              this.downloadModeling(item) // 다운로드가 완료될 때까지 대기
              await this.clickComponent(item, media) // clickComponent 함수가 끝날 때까지 대기
            }
          }
        }
      }
    } catch (error) {
      console.error('Error fetching data:', error)
    }
  },
  methods: {
    handleClick() {
      if (this.tab === 0) {
        this.mappingNext()
      } else {
        this.mappingAdd()
      }
    },
    sendIson(isOn, child) {
      this.isOn = isOn
      this.$emit('send-ison', this.isOn, child)
    },
    sendLockIson(isOn, child) {
      this.$emit('send-lock-ison', isOn, child)
    },
    sendLinkIson(isOn, child) {
      this.isMappingOn = true
    },
    handleMenuItemClick(id, root = false) {
      if (root) this.nowClickListId = id
      this.setClickObj = this.setClickObj.filter(item => item.id !== this.nowClickListId)
      this.$emit('resetting-moedling-list', this.nowClickListId)

      this.showMenu = false
    },
    showContextMenu(event, id) {
      this.menuX = event.clientX
      this.menuY = event.clientY
      this.showMenu = true
      this.nowClickListId = id
    },
    removePrimitiveSuffix(name) {
      const suffix = '_primitive'
      const index = name.indexOf(suffix)
      if (index !== -1) {
        return name.substring(0, index)
      }
      return name
    },
    getBackgroundColor(name) {
      if (this.treeClickId === name) {
        return '#E0E0E0'
      } else if (this.hoverChildId === name) {
        return '#F8F8F9'
      }
      return ''
    },
    mappingReset(newVal) {
      for (let i = 0; i < this.meshMappingList.length; i++) {
        if (this.meshMappingList[i].meshid === this.treeClickId) {
          this.mappingClickId = ''
          this.meshMappingList.splice(i, 1) // 요소 삭제
          this.nowMappingChild.isMapping = false
          break // 삭제 후 루프 종료 (필요한 경우)
        }
      }
      this.$emit('now-setting-mapping', this.meshMappingList)
    },
    mappingNext(newVal) {
      if (this.mappingClickId != '' && this.treeClickId != '') {
        this.tab = 1
      }
    },
    mappingAdd(newVal) {
      this.tab = 0
      this.isMappingOn = false
      this.nowMappingChild.isMapping = true
      const isIN = this.meshMappingList.some(item => item.meshid === this.treeClickId) // 이미 매핑됐는지 확인

      if (isIN) {
        // 이미 매핑됐을 경우
        for (let i = 0; i < this.meshMappingList.length; i++) {
          if (this.meshMappingList[i].meshid === this.treeClickId) {
            this.$set(this.meshMappingList[i], 'deviceid', this.mappingClickId)
          }
        }
      } else {
        // 새로 매핑 추가
        if (this.treeClickId) {
          this.meshMappingList.push({
            meshid: this.treeClickId,
            deviceid: this.mappingClickId,
          }) // 메쉬id, 장치id
        }
      }
      this.$emit('now-setting-mapping', this.meshMappingList)
    },
    clickDeviceId(newVal) {
      this.mappingClickId = newVal
      //id매핑 리스트 클릭시 메쉬, 장치 연결
      // if (this.treeClickId) {
      //   //레이어 클릭 했는지 확인
      //   const isIN = Object.values(this.meshMappingList).some(item => item.meshid === this.treeClickId) //이미 매핑 됐는지 확인
      //   if (isIN) {
      //     //이미 매핑됐을 경우
      //     for (const key in this.meshMappingList) {
      //       if (this.meshMappingList[key].meshid === this.treeClickId) {
      //         this.$set(this.meshMappingList[key], 'deviceid', newVal)
      //       }
      //     }
      //   } else {
      //     //새로 매핑 추가
      //     if (this.treeClickId) {
      //       const generateId = this.generateRandomId()
      //       this.$set(this.meshMappingList, generateId, {
      //         meshid: this.treeClickId,
      //         deviceid: newVal,
      //       }) //메쉬id, 장치id
      //       this.meshMappingClickNum = generateId
      //     }
      //   }
      // }
    },

    childTreeClick(child) {
      //.id, child.createId, child.meshId
      this.nowClickListId = child.id
      //this.selectedChildId = child.id
      this.treeClickId = child.id
      //this.$refs.babylonCanvas.selectMesh(this.selectedChildId)
      if (child.createId != this.nowMappingChild.createId) {
        this.$emit('now-setting-mapping', this.meshMappingList)
        //this.meshMappingList = []
        this.nowMappingClickTemp = child.createId
        this.$emit('now-click-temp', child.createId)
      }
      this.nowMappingChild = child

      //this.$emit('tree-click-id', this.selectedChildId, childCreateId, meshId)
    },
    changeMappingData(newVal) {
      this.meshMappingList = newVal
    },
    sendMeshMapping() {
      this.$emit('now-setting-mapping', this.meshMappingList)
    },
    generateRandomId() {
      return 'id-' + Math.random().toString(36).substr(2, 9) + '-' + Date.now()
    },
    handleTreeViewClick(event) {
      event.stopPropagation()
    },
    async clickComponent(item, media = '') {
      this.modalListCheck = true
      await new Promise((resolve, reject) => {
        this.$emit('click-component', item, media, resolve) // resolve를 전달
      })
    },
    resetComponent() {
      this.modalListCheck = false
    },
    truncateFilename(name) {
      const maxLength = 10
      if (name.length > maxLength) {
        return name.substring(0, maxLength) + '...'
      } else {
        return name
      }
    },

    parseTags(tags) {
      try {
        return JSON.parse(tags)
      } catch (e) {
        console.error('Error parsing tags:', e)
        return [] // 파싱 실패 시 빈 배열 반환
      }
    },
    checkModalTagIn(item, getOptionList) {
      if (Object.keys(getOptionList).length === 0) return true
      if (item.properties.tag) {
        const optionValues = Object.values(getOptionList).reduce((acc, option) => {
          if (option.label) {
            acc.push(option.label)
          }
          return acc
        }, [])
        const hasCommonTags = item.properties.tag.some(tag => optionValues.includes(tag))
        return hasCommonTags
      } else {
        return true
      }
    },
    getModalSelectOption(data) {
      this.getModalSelectedOption = data
    },
    selectItem(index) {
      this.selectedItem = index
    },
    downloadModeling(item) {
      if (!item.downloaded) {
        item.downloaded = true
        item.onoff = true
        this.downloadModelingList.push(item)
      }
    },
    searchToOn() {
      this.clickSearch = true
    },
    convertBlobToBase64(blob) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader()
        reader.onerror = reject
        reader.onload = () => {
          resolve(reader.result) // 결과는 data:url 형식의 Base64 문자열
        }
        reader.readAsDataURL(blob)
      })
    },
    async fetchImageAsBase64(filename, path) {
      try {
        const response = await this.$axios.get(`${this.gractorBaseUrl}/api/media/download`, {
          params: {
            filename: filename, // 파일 이름
            path: path, // 파일 경로
          },
          responseType: 'blob', // 중요: 응답을 Blob으로 받기 위해 설정
        })
        // Blob 응답을 Base64로 변환
        return this.convertBlobToBase64(response.data)
      } catch (error) {
        console.error('Error fetching the image:', error)
      }
    },
  },
}
</script>

<style>
/* @media (max-width: 2560px) {
  .layer_div {
    overflow: auto;
    max-height: 1200px;
  }
}
@media (max-width: 1920px) {
  .layer_div {
    overflow: auto;
    max-height: 600px;
  }
} */
.media-download-modal {
  box-shadow: 0px 3.773px 8.803px 0px rgba(0, 0, 0, 0.15);
  border-radius: 0.89821rem;
  padding: 1.64rem 2.14rem 1.64rem 0rem;
  /* height: 40.92857rem; */
}
.media-download-modal-left {
  width: 54%;
  padding: 2.79rem 6.14rem 0rem 7.14rem;
}
.modalview-layer {
  padding: 0.875rem;
  border-radius: 0.35714rem;
}

.modal-title-download {
  background-color: rgba(240, 238, 255, 1) !important;
  color: rgba(126, 115, 254, 1) !important;
  padding-left: 4.79rem !important;
  padding-right: 6.21rem !important;
  right: 0;
  bottom: 0;
}
.media-download-modal-tag {
  gap: 0.69rem;
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
}

.media-download-modal-img {
  height: 22.42857rem;
  margin-bottom: 1.79rem;
}
.media-modal-img-main {
  height: 22.42857rem;
  border-radius: 0.312rem;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}
.media-modal-img-main-img {
  max-width: 100%;
  max-height: 100%;
  background-color: rgb(255, 255, 255); /* 이미지가 차지하지 않는 부분을 검은색으로 채움 */
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}
.media-download-modal-line {
  width: 0.2rem;
  height: 90%;
  background-color: #f0f0f0;
}
.media-download-modal-right {
  padding-left: 6.14rem;
}
.media-modal-info-title {
  color: #434445;
  font-size: 0.8125rem;
  width: 6.69rem;
}
.media-modal-info-detail {
  color: #848586;
  font-size: 0.8125rem;
}

.media-download-modal-close {
  margin-bottom: 0.86rem;
  width: 95%;
}
.media-download-model-list {
  height: 80%;
  overflow-x: hidden;
  overflow-y: auto;
  /* max-height: 70vh; */
}
.media-download-model-list::-webkit-scrollbar {
  width: 0.4rem;
}
.media-download-model-list::-webkit-scrollbar-thumb {
  background-color: #d5d5d5;
  border-radius: 10px;
}
.media-download-model-list::-webkit-scrollbar-track {
  background: #fff;
  border-radius: 10px;
}
.selected-item {
  /* border: 1.8px solid var(--cms-purple-002, #a8a1ff) !important; */
  background-color: #f0f0f0;
}
.media-download-model-list-img {
  width: 7rem;
  height: 5rem;
  border-radius: 0.312rem;
  margin-right: 1.19rem;
  background-color: #f8f8f9;
}

.builder-side-box-midea .v-treeview-node__toggle {
  transform: none;
}

.root-node {
  cursor: pointer;
  align-items: center;
}
.root-node .v-list-group__header {
  background-color: #f8f8f9;
  width: 100%;
  padding: 0 !important;
  align-items: center;
  display: flex !important;
  min-height: 2.857rem;
  border-radius: 0.21rem;
}
.root-node .v-list-item__icon {
  margin: 0 !important;
  padding: 0.43rem 0.43rem 0.43rem 0.86rem !important;
  /* width: 3.29rem;
  height: 2.86rem; */
}
.tree-node .v-list-group__items {
  width: 100%;
}
.root-node .v-list-group__header__prepend-icon {
  /* padding: 12px 0.43rem 12px 0.86rem !important; */
}
.child-node {
  padding: 0 0 0 0.71rem !important;
  min-height: 2rem !important;
  margin-bottom: 0.14rem;
  border-radius: 0.21429rem;
}
.children {
  overflow: hidden;
  margin-left: 0.43rem;
  margin-top: 0.29rem;
}
</style>
