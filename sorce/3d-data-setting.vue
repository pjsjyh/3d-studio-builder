<template>
  <section class="f-row w-100 f-full h-100" style="flex: 1">
    <v-expansion-panels v-show="sideboxSelected === 'layer'" v-model="panel" style="width: 20.28571rem; pointer-events: all; height: fit-content" multiple>
      <!-- <v-expansion-panel>
        <v-expansion-panel-header style="color: #4136c0; font-weight: 600">메시 데이터 매핑 리스트</v-expansion-panel-header>
        <v-expansion-panel-content
          style="display: flex; justify-content: center; align-items: center; height: 14.21rem"
          :style="{
            justifyContent: meshMappingListLengthZero ? 'center' : 'normal',
            alignItems: meshMappingListLengthZero ? 'center' : 'normal',
          }"
        >
          <MediaListEmpty
            :style="{ display: meshMappingListLengthZero ? 'block' : 'none !important' }"
            :get-word="`매핑된 메시가 없습니다.\n 메시에 ID값을 매핑해 주세요.`"
            :get-img="`mdi-database-outline`"
            :img-size="'2.85rem'"
            :is-visible="false"
          ></MediaListEmpty>
          <div class="w-100 h-100" style="overflow-y: scroll; overflow-x: hidden" :style="{ display: meshMappingListLengthZero ? 'none' : 'block' }">
            <tbody class="w-100 d-flex flex-column">
              <tr
                v-for="(item, key) in meshMappingList"
                :key="key"
                class="w-100 justify-content-between d-flex"
                style="cursor: pointer; border-radius: 0.21429rem; padding: 0.71rem 0.57rem 0.71rem 0.5rem"
                :style="{ backgroundColor: meshMappingClickNum === key ? '#f8f8f9' : '' }"
                @click="meshMappingClick(item, key)"
              >
                <td>{{ truncateFilename(findMeshName(item.meshid)) }}</td>
                <td style="color: #9c9c9c" @click="deleteMapplingClick(key, meshMappingList)">삭제</td>
              </tr>
            </tbody>
          </div>
        </v-expansion-panel-content>
      </v-expansion-panel> -->
      <v-expansion-panel>
        <v-expansion-panel-header style="color: #4136c0; font-weight: 600">애니메이션 리스트</v-expansion-panel-header>
        <v-expansion-panel-content class="animationlist-content" style="display: flex; justify-content: center; height: 14.21rem">
          <v-btn-toggle
            v-model="animationSelect"
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
              >내장 애니메이션</v-btn
            >
            <v-btn
              class="animation-btn-small"
              value="scene"
              style="font-size: 0.85714rem"
              :style="{
                fontWeight: animationSelect === 'scene' ? '600' : '400',
              }"
              >오브젝트 애니메이션</v-btn
            >
          </v-btn-toggle>
          <div class="w-100 h-100 no-select" style="overflow-y: scroll; overflow-x: hidden; flex: 1">
            <tbody v-if="animationSelect === 'mesh'" class="w-100 d-flex flex-column h-100">
              <MediaListEmpty
                :style="{ display: settingModelingList.length === 0 ? 'block' : 'none !important' }"
                :get-word="`내장 애니메이션이 없습니다.`"
                :get-img="``"
                :img-size="'2.85rem'"
                :is-visible="false"
              ></MediaListEmpty>
              <v-list :style="{ display: settingModelingList.length != 0 ? 'block' : 'none !important' }">
                <v-list-item style="overflow-y: auto; position: inherit; overflow-x: hidden; display: flex; flex-direction: column">
                  <div v-for="(item, index) in settingModelingList" :key="index" class="w-100 mb-2">
                    <div class="d-flex align-items-center justify-content-between pl-2 pt-2 pb-2">애니메이션 {{ index + 1 }}</div>
                    <div class="d-flex align-items-center justify-content-between pl-4">
                      {{ settingModelingAniName[item.animationIndex] }}
                      <v-icon :class="item.disable[0] ? 'dti-eye-open' : 'dti-eye-close'"></v-icon>
                    </div>
                  </div>
                </v-list-item>
              </v-list>
            </tbody>
            <template v-else-if="animationSelect === 'scene'">
              <MediaListEmpty
                :style="{ display: makeSceneAniSet.length === 0 ? 'block' : 'none !important' }"
                :get-word="`씬 애니메이션이 없습니다.`"
                :get-img="``"
                :img-size="'2.85rem'"
                :is-visible="false"
              ></MediaListEmpty>
              <thead>
                <tr>
                  <th class="text-left">오브젝트</th>
                  <th class="text-left">애니메이션 유형</th>
                </tr>
              </thead>
              <tbody class="w-100 d-flex flex-column h-100">
                <tr
                  v-for="(item, key) in makeSceneAniSet"
                  :key="key"
                  class="w-100 justify-content-between d-flex"
                  style="cursor: pointer; border-radius: 0.21429rem; padding: 0.71rem 0.57rem 0.71rem 0.5rem"
                  :style="{
                    backgroundColor: aniMappingClickNum.includes(key) ? '#f8f8f9' : '',
                    display: makeSceneAniSet.length != 0 ? 'block' : 'none !important',
                  }"
                  @click="meshMappingClick(item, key)"
                >
                  <td>'-'</td>
                  <td>{{ item.animationType }}</td>
                  <td style="color: #9c9c9c" @click="deleteMapplingClick(item.sceneAniTempatedId)">삭제</td>
                </tr>
              </tbody>
            </template>
          </div>
        </v-expansion-panel-content>
      </v-expansion-panel>
      <v-expansion-panel>
        <v-expansion-panel-header style="color: #4136c0; font-weight: 600">이벤트 설정 리스트</v-expansion-panel-header>
        <v-expansion-panel-content style="display: flex; justify-content: center; align-items: center; height: 14.21rem">
          <MediaListEmpty :get-word="``" :get-img="``" :img-size="'2.85rem'" :is-visible="false"></MediaListEmpty>
          <DataList></DataList>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>
    <div class="h-100" style="display: flex; align-items: center; pointer-events: all">
      <div class="data-template-box-right-btn" :style="{ backgroundColor: darkMode ? '#1F1F1F' : '#fff' }" @click="dataBoxClose = !dataBoxClose">
        <div>
          <v-icon style="width: 2rem; height: 2rem">{{ dataBoxClose ? 'mdi-chevron-right' : 'mdi-chevron-left' }}</v-icon>
        </div>
      </div>
      <div style="width: 26.57143rem; height: 100%">
        <div
          v-show="sideboxSelected === 'media'"
          class="h-100 bg-white"
          style="
            padding: 1.93rem 1.43rem;
            box-shadow: 0px 3px 1px -2px rgba(0, 0, 0, 0.2), 0px 2px 2px 0px rgba(0, 0, 0, 0.14), 0px 1px 5px 0px rgba(0, 0, 0, 0.12);
          "
        >
          <span style="font-weight: 600; color: #4136c0; height: 2.42857rem; display: flex; align-items: center">환경 설정</span>
        </div>
        <div v-show="sideboxSelected === 'layer'" class="h-100 d-flex flex-column bg-white">
          <v-tabs v-model="tab" style="flex: none; padding: 0 2rem">
            <v-tab style="flex: 1" value="0">애니메이션</v-tab>
            <v-tab style="flex: 1" value="1">이벤트 설정</v-tab>
          </v-tabs>
          <v-tabs-items v-model="tab" style="flex: 1; padding: 2.14rem 2rem">
            <v-tab-item>
              <v-card flat class="d-flex flex-column">
                <AnimationMapping
                  ref="aniMapping"
                  :setting-modeling-ani-name="settingModelingAniName"
                  :ani-mapping-click-num="aniMappingClickNum"
                  :object-ani-click-item="objectAniClickItem"
                  @make-scene-ani="makeSceneAni"
                  @now-setting-scene-ani="nowSettingSeceneAni"
                ></AnimationMapping>
              </v-card>
            </v-tab-item>
            <v-tab-item>
              <v-card flat class="d-flex flex-column"></v-card>
            </v-tab-item>
          </v-tabs-items>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  components: {
    AnimationMapping: () => import('~/components/viewtrack/source/3d-animation-mapping.vue'),
    MediaListEmpty: () => import('~/components/viewtrack/source/media-list-empty.vue'),

    DataList: () => import('~/components/viewtrack/source/3d-data-list.vue'),
  },
  props: {
    sideboxSelected: { type: String, default: 'media' },
    settingModelingList: { type: Array, default: () => [] },
    settingModelingAniName: { type: Array, default: () => [] },
    aniMappingClickNum: { type: Array, default: () => [] },
  },
  data() {
    return {
      tab: 0,
      dataStepper: 1,
      panel: [0],
      animationSelect: 'mesh',

      saveMeshAniSet: [], //메시애니 데이터
      animationMappingList: {}, //애니메이션 매핑 리스트
      dataBoxClose: true, //오른쪽 box onoff
      aniMappingClickNum2: [],

      saveSceneAniSet: [], //저장할 애니 데이터
      makeSceneAniSet: [],

      objectAniClickItem: {},
    }
  },
  computed: {
    darkMode() {
      return this.$vuetify.theme.dark
    },
  },
  watch: {
    tab(newVal) {
      this.panel = []
      this.panel.push(newVal)
    },
    dataBoxClose(newVal) {
      this.$emit('data-box-close', newVal)
    },
  },
  methods: {
    nowSettingSeceneAni(newVal) {
      this.$emit('now-setting-scene-ani', newVal)
    },
    meshMappingClick(item, key) {
      //클릭하면 배경색 바뀌에 번호 저장
      //console.info(item, key)
      this.objectAniClickItem = item
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
    deleteMapplingClick(getkey) {
      //매핑 삭제
      //this.$aniMapping.test(getkey)
      console.info(this.makeSceneAniSet)
      if (Object.keys(this.makeSceneAniSet).length != 0) {
        const isIn = this.makeSceneAniSet.find(ani => ani.sceneAniTempatedId === getkey)
        if (isIn) {
          const index = this.makeSceneAniSet.indexOf(isIn)
          if (index !== -1) {
            // 해당 항목을 리스트에서 삭제
            this.$refs.aniMapping.deleteAniList(getkey)
            this.$emit('delete-ani', getkey)
            console.info(this.makeSceneAniSet, this.makeSceneAniSet.length)
            //let removedElement = this.makeSceneAniSet.splice(index, 1)

            //console.info(removedElement)
            console.info(this.makeSceneAniSet, index)
            //리스트 삭제가 안됨
          }
        }
      }
      console.info(this.makeSceneAniSet)
    },
    makeAniList(newVal) {
      //애니메이션 리스트 추가
      //this.animationMappingList = newVal
      this.saveMeshAniSet = newVal
      this.$set(this, 'saveMeshAniSet', newVal)
    },
    makeSceneAni(newVal) {
      this.makeSceneAniSet = newVal
    },
  },
}
</script>
