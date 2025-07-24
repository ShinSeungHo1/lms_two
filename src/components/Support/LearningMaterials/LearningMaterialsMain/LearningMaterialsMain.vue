<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import LearningMaterialsModal from '../LearningMaterialsModal/LearningMaterialsModal.vue';
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();

const modalState = useModalState();

const mtrList = ref([]);
const mtrCnt = ref(0);
const lectures = ref([]);

const mtrIdForDetail = ref(0);
const lecIdForDetail = ref(0);
const lecNameForDetail = ref('');

const handlerSearchMtrList = (cPage = 1) => {
  const params = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(params);

  axios.post('/api/support/getMtrListBody.do', urlParam).then((res) => {
    console.log('MtrList 불러오기 성공!');
    console.log(res);
    mtrList.value = res.data.mtrList;
    mtrCnt.value = res.data.mtrCnt;
    lectures.value = res.data.lectures;
  });
};

const handlerDetailModal = (mtrId, lecId, lecName) => {
  modalState.$patch({ isOpen: true, type: 'detailMtr' });
  mtrIdForDetail.value = mtrId;
  lecIdForDetail.value = lecId;
  lecNameForDetail.value = lecName;
};

watch(
  () => route.query,
  () => {
    handlerSearchMtrList();
  },
);

onMounted(() => {
  handlerSearchMtrList();
});
</script>

<template>
  <div class="learningMaterials-main-container">
    <table class="learningMaterials-table">
      <thead class="learningMaterials-table-header">
        <tr>
          <th>강의명</th>
          <th>작성자</th>
          <th>제목</th>
          <th>작성일</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="mtrCnt > 0">
          <tr v-for="mtr in mtrList" :key="mtr.materiId" class="learningMaterials-table-row">
            <td class="learningMaterials-cell">{{ mtr.lecName }}</td>
            <td class="learningMaterials-cell">{{ mtr.loginId }}</td>
            <td
              class="learningMaterials-cell cursor-pointer hover:underline"
              @click="handlerDetailModal(mtr.materiId, mtr.lecId, mtr.lecName)"
            >
              {{ mtr.materiTitle }}
            </td>
            <td class="learningMaterials-cell">{{ mtr.materiDate }}</td>
          </tr>
        </template>
        <template v-else>
          <tr>
            <td colspan="4" class="learningMaterials-empty-row">데이터가 존재하지 않습니다.</td>
          </tr>
        </template>
      </tbody>
    </table>
    <PageNavigation
      :total-items="mtrCnt"
      :items-per-page="5"
      :on-page-change="handlerSearchMtrList"
    />
  </div>
  <LearningMaterialsModal
    v-if="modalState.type === 'detailMtr' && modalState.isOpen"
    :mtr-id="mtrIdForDetail"
    :lec-id="lecIdForDetail"
    :lec-name="lecNameForDetail"
    :lecture-list="lectures"
    @success="handlerSearchMtrList"
  />
  <LearningMaterialsModal
    v-if="modalState.type === 'createMtr' && modalState.isOpen"
    :lecture-list="lectures"
    @success="handlerSearchMtrList"
  />
</template>

<style>
@import './styled.css';
</style>
