<template>
  <div class="justify-content-center d-flex flex-column align-items-center" style="width: 22.57rem">
    <div style="gap: 1rem; display: grid">
      <div class="d-flex" style="gap: 0.86rem">
        <div style="width: 9.28571rem">
          <TemplateFormFieldVuetify
            :custom-field="true"
            :field-model="{
              type: 'select',
              items: typeOption,
            }"
            :bind-value="{
              clearable: true,
              'hide-details': true,
              dense: true,
              outlined: true,
              placeholder: '타입',
              color: 'deep-purple accent-2',
            }"
            @input="getMeshType"
          />
        </div>

        <TemplateFormFieldVuetify
          style="width: 12.57143rem"
          :custom-field="true"
          :field-model="{
            type: 'select',
            items: mediaSrcFilterOption,
          }"
          :bind-value="{
            clearable: true,
            'hide-details': true,
            dense: true,
            outlined: true,
            placeholder: '세부 사항',
            color: 'deep-purple accent-2',
          }"
        />
      </div>
      <div class="d-flex align-items-baseline justify-content-between">
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
            label: '검색',
            clearable: true,
            color: 'deep-purple accent-2',
            'show-size': true,
            'truncate-length': 15,
          }"
          @input="
            value => {
              searchQuery = value
            }
          "
        />
        <v-btn style="width: 5.42857rem; background-color: var(--DT-light-dt_bg_3, #f8f8f9)" depressed @click="performSearch">조회</v-btn>
      </div>
    </div>
    <div style="margin-top: 2.57rem; width: 22.57rem" class="mapping-data">
      <v-data-table
        :headers="headers"
        :items="computedDataList"
        :page.sync="page"
        :items-per-page="itemsPerPage"
        hide-default-footer
        :search="activeSearch"
        :custom-filter="customFilter"
        @page-count="pageCount = $event"
      >
        <template #item="{ item }">
          <tr class="g-cursor-pointer" :style="{ backgroundColor: dataclickId === item.facilityId ? '#f8f8f9' : '' }" @click="clickData(item.facilityId)">
            <td>{{ item.no }}</td>
            <td>{{ item.facilityName }}</td>
            <td>{{ item.facilityId }}</td>
          </tr>
        </template></v-data-table
      >
      <div class="text-center pt-2">
        <v-pagination
          v-model="page"
          class="custom-pagination"
          :length="pageCount"
          circle
          color="var(--DT-light-dt_primary_active, rgba(126, 115, 254, 0.16))"
        ></v-pagination>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  components: {
    TemplateFormFieldVuetify: () => import('~/components/template/template-form-field-vuetify.vue'),
  },
  data() {
    return {
      search: '',
      mediaSrcFilterOption: [{ text: '전체보기', value: 'all' }],
      typeOption: [
        { text: '장치', value: 'device' },
        { text: 'facility', value: 'facility' },
        { text: '공간', value: 'space' },
      ],

      headers: [
        {
          text: 'No',
          align: 'start',
          sortable: false,
          value: 'number',
        },
        { text: '이름', value: 'name' },
        { text: '장치 ID', value: 'facilityId' },
      ],
      desserts: [
        {
          facilityName: 'Frozen Yogurt',
          facilityId: 1,
        },
        {
          facilityName: 'Ice cream sandwich',
          facilityId: 2,
        },
        {
          facilityName: 'Eclair',
          facilityId: 3,
        },
        {
          facilityName: 'Cupcake',
          facilityId: 4,
        },
        {
          facilityName: 'Gingerbread',
          facilityId: 5,
        },
        {
          facilityName: 'Jelly bean',
          facilityId: 6,
        },
        {
          facilityName: 'Lollipop',
          facilityId: 7,
        },
        {
          facilityName: 'Honeycomb',
          facilityId: 8,
        },
        {
          facilityName: 'Donut',
          facilityId: 9,
        },
        {
          facilityName: 'KitKat',
          facilityId: 10,
        },
      ],

      dataList: [],
      page: 1,
      pageCount: 0,
      itemsPerPage: 5,
      searchQuery: '',
      activeSearch: '',

      getType: '',
      getDetail: '',

      dataclickId: '',
    }
  },
  computed: {
    computedDataList() {
      return this.dataList.length === 0 ? this.desserts : this.dataList
    },
  },
  async mounted() {
    // const response = await this.$axios.get(`/api/devices?serviceId=_VxeH4wB9Ttn5FLKFfDb`)
    // console.info(response)

    // const aaa = await this.$axios
    //   .get('/api/devices')
    //   .then(response => {
    //     if (response.status === 200) {
    //       const rows = response.data.rows
    //       if (Array.isArray(rows)) {
    //         rows.forEach(row => {
    //           console.info(row.docType)
    //         })
    //       } else {
    //         console.log('rows가 배열이 아닙니다:', rows)
    //       }
    //     } else {
    //       console.log('요청은 성공했지만 예상하지 못한 상태 코드:', response.status)
    //     }
    //   })
    //   .catch(error => {
    //     console.error('API 요청 중 오류 발생:', error)
    //   })
    this.updateItemsPerPage()
    window.addEventListener('resize', this.updateItemsPerPage)
    const { data: { rows: controlDevices = [] } = {} } = await this.$axios.get('/api/devices', {
      params: {
        serviceId: '_VxeH4wB9Ttn5FLKFfDb',
      },
    })
    this.dataList = controlDevices
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.updateItemsPerPage)
  },
  methods: {
    updateItemsPerPage() {
      if (window.innerHeight < 950) {
        this.itemsPerPage = 5
      } else {
        this.itemsPerPage = 10
      }
    },
    customFilter(value, search, item) {
      return Object.values(item).some(val => String(val).toLowerCase().includes(search.toLowerCase()))
    },
    performSearch() {
      this.activeSearch = this.searchQuery
    },
    clickData(data) {
      this.dataclickId = data
      this.$emit('click-device-id', data)
    },
    getMeshType(newVal) {
      this.getType = newVal
    },
  },
}
</script>

<style>
.custom-pagination .v-pagination__item,
.custom-pagination .v-pagination__navigation {
  min-width: 24px; /* 최소 너비를 줄입니다 */
  height: 24px; /* 높이를 줄입니다 */
  line-height: 24px; /* 텍스트가 중앙에 오도록 조정합니다 */
  font-size: 12px; /* 폰트 크기를 줄입니다 */
  padding: 0; /* 여백을 제거합니다 */
  margin: 0 2px; /* 페이지 번호 간격을 최소화합니다 */
  border: none; /* 테두리를 제거 */
  box-shadow: none; /* 그림자를 제거 */
  background-color: transparent; /* 배경색을 투명하게 */
}

.custom-pagination .v-pagination__more {
  min-width: 24px; /* ... 버튼의 최소 너비를 줄입니다 */
  height: 24px; /* 높이를 줄입니다 */
  line-height: 24px; /* 텍스트가 중앙에 오도록 조정합니다 */
  font-size: 12px; /* 폰트 크기를 줄입니다 */
  padding: 0; /* 여백을 제거합니다 */
  margin: 0 2px; /* ... 버튼 간격을 최소화합니다 */
  border: none;
  box-shadow: none;
  background-color: transparent;
}
.custom-pagination .theme--light.v-pagination .v-pagination__item--active {
  color: #000;
}
.mapping-data .v-data-table__wrapper > table > thead > tr > th {
  height: 1.42875rem;
}
.mapping-data .v-data-table > .v-data-table__wrapper > table > tbody > tr > td {
  height: 2.214rem;
}
</style>
