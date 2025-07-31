<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import TestInfoDetailModal from '../TestInfoDetailModal/TestInfoDetailModal.vue';

import axios from 'axios';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';
import { useModalState } from '@/stores/modalState';
import { useUserInfo } from '@/stores/loginInfoState';

const { user } = useUserInfo();
const route = useRoute();
const modalState = useModalState();

const headList = [
  '강의',
  '강사',
  '강의실',
  '시험 시작일',
  '시험 종료일',
  '시험 등록일',
  '시험문제보기',
];

const testInfoId = ref(0);
const testLecId = ref(0);

const testInfoList = ref([]);
const listCount = ref(0);

const searchTestInfo = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/lecture/testInfoListBody.do', urlParam)
    .then((res) => {
      console.log('Test-Info 불러오기 성공!');
      testInfoList.value = res.data.list;
      listCount.value = res.data.count;
    })
    .catch((err) => {
      console.log('Test-Info 불러오기 서버 오류 발생 err : ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const showTestInfoDetailModal = (testId, lecId) => {
  modalState.$patch({ isOpen: true, type: 'testInfoDetail' });
  testInfoId.value = testId;
  testLecId.value = lecId;
};

onMounted(() => {
  searchTestInfo();
});
</script>
<template>
  <div class="test-info-table-container">
    <table class="test-info-table">
      <thead class="test-info-table-head">
        <tr class="test-info-table-head-row">
          <th
            v-for="(head, index) in headList"
            :key="`h-${index}`"
            class="test-info-table-head-cell"
          >
            {{ head }}
          </th>
        </tr>
      </thead>
      <tbody class="test-info-table-body">
        <tr
          v-for="testInfo in testInfoList"
          :key="testInfo.testId"
          class="test-info-table-body-row"
        >
          <th class="test-info-table-body-cell">{{ testInfo.lecName }}</th>
          <th class="test-info-table-body-cell">{{ testInfo.lecInstructorName }}</th>
          <th class="test-info-table-body-cell">{{ testInfo.lecRoomName }}</th>
          <th class="test-info-table-body-cell">{{ testInfo.testBeginDate?.substring(0, 10) }}</th>
          <th class="test-info-table-body-cell">{{ testInfo.testEndDate?.substring(0, 10) }}</th>
          <th class="test-info-table-body-cell">{{ testInfo.testRegDate?.substring(0, 10) }}</th>
          <th class="test-info-table-body-cell">
            <button
              class="test-info-btn"
              @click="showTestInfoDetailModal(testInfo.testId, testInfo.lecId)"
            >
              시험문제보기
            </button>
          </th>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation :items-per-page="5" :total-items="listCount" :on-page-change="searchTestInfo" />
  <TestInfoDetailModal
    v-if="modalState.isOpen && modalState.type === 'testInfoDetail' && user.userType === 'T'"
    :test-id="testInfoId"
    :lec-id="testLecId"
  />
</template>
<style>
@import './styled.css';
</style>
