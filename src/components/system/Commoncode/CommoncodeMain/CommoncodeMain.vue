<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();
const modalState = useModalState();

const commonList = ref([]);
const commonCount = ref(0);

const headerList = ['대분류', '그룹코드', '상세코드', '상세명', '번호'];

const commonSearch = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);

  axios.post('/api/system/commonListJson.do', urlParam).then((res) => {
    console.log('공통 코드 리스트 불러오기 성공!');
    commonCount.value = res.data.CommonCnt;
    commonList.value = res.data.list;
  });
};

const showCommonDetail = () => {
  modalState.$patch({ isOpen: true, type: 'commonDetail' });
};

watch(
  () => route.query,
  () => {
    commonSearch();
  },
);

onMounted(() => {
  commonSearch();
});
</script>
<template>
  <div class="table-container">
    <table class="table">
      <thead class="table-header">
        <tr class="table-header-row">
          <th v-for="(header, index) in headerList" :key="`h-${index}`" class="table-header-cell">
            {{ header }}
          </th>
        </tr>
      </thead>
      <tbody class="table-body">
        <tr v-for="item in commonList" :key="item.detailCode" class="table-body-row">
          <td class="table-body-cell">{{ item.useYn === 'equip' ? '장비' : '지역' }}</td>
          <td class="table-body-cell">{{ item.groupName }}</td>
          <td class="table-body-cell">{{ item.detailCode }}</td>
          <td class="table-body-cell" @click="showCommonDetail">{{ item.detailName }}</td>
          <td class="table-body-cell">{{ item.regId }}</td>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation :items-per-page="5" :total-items="commonCount" :on-page-change="commonSearch" />
  <CommonCodeModal />
</template>
<style>
@import './styled.css';
</style>
