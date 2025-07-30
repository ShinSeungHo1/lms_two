<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import TestModal from '../TestModal/TestModal.vue';

import axios from 'axios';
import { useUserInfo } from '@/stores/loginInfoState';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';
import { useModalState } from '@/stores/modalState';

const headList = [
  '강의',
  '강사',
  '강의실',
  '시험 시작일',
  '시험 종료일',
  '시험응시',
  '시험응시결과',
];

const { user } = useUserInfo();
const route = useRoute();
const modalState = useModalState();

const testList = ref([]);
const listCount = ref(0);

const lectureId = ref(0);
const lecTestId = ref(0);
const lecStudentId = ref('');

const openTestModal = (lecId, testId, studentId, payload) => {
  modalState.$patch({ isOpen: true, type: 'testResult', payload: payload });
  lectureId.value = lecId;
  lecTestId.value = testId;
  lecStudentId.value = studentId;
};

const disableTestBtn = (testAvailable, submitYn) => {
  if (testAvailable && submitYn === 'N') return false;
  if (testAvailable && submitYn === 'Y') return false;
  if (!testAvailable && submitYn === 'Y') return true;
  if (!testAvailable && submitYn === 'N') return true;
};

const disableTestResultBtn = (testAvailable, scoreYn) => {
  if (testAvailable && scoreYn === 'N') return true;
  if (testAvailable && scoreYn === 'Y') return false;
  if (!testAvailable && scoreYn === 'Y') return false;
  if (!testAvailable && scoreYn === 'N') return true;
};

const testStatus = (testAvailable, submitYn) => {
  if (testAvailable && submitYn === 'N') {
    return '시험응시';
  }

  if (testAvailable && submitYn === 'Y') {
    return '시험응시완료';
  }

  if (!testAvailable && submitYn === 'Y') {
    return '시험응시종료';
  }

  if (!testAvailable && submitYn === 'N') {
    return '시험미응시';
  }
};

const testResultStatus = (testAvailable, scoreYn) => {
  if (testAvailable && scoreYn === 'N') {
    return '시험응시예정';
  }

  if (testAvailable && scoreYn === 'Y') {
    return '시험응시결과';
  }

  if (!testAvailable && scoreYn === 'Y') {
    return '시험종료';
  }

  if (!testAvailable && scoreYn === 'N') {
    return '시험미응시';
  }
};

const searchTest = (cPage = 1) => {
  const param = {
    studentId: user.loginId,
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/lecture/testListBody.do', urlParam)
    .then((res) => {
      console.log(res);
      console.log('학생 테스트 리스트 불러오기 성공');
      testList.value = res.data.list;
      listCount.value = res.data.count;
    })
    .catch((err) => {
      console.log('학생 테스트 리스트 불러오기 서버 오류 발생 err : ', err);
      ElMessage.error('서버 오류 발생');
    });
};

watch(
  () => route.query,
  () => {
    searchTest();
  },
);

onMounted(() => {
  searchTest();
});
</script>
<template>
  <div class="std-test-container">
    <div class="std-test-container-contnet">
      <div class="std-test-table-container">
        <table class="std-test-table">
          <thead class="std-test-table-head">
            <tr class="std-test-table-head-row">
              <th
                v-for="(head, index) in headList"
                :key="`h-${index}`"
                class="std-test-table-head-cell"
              >
                {{ head }}
              </th>
            </tr>
          </thead>
          <tbody class="std-test-table-body">
            <tr v-for="test in testList" :key="test.testId" class="std-test-table-body-row">
              <th class="std-test-table-body-cell">
                {{ test.lecName }}
              </th>
              <th class="std-test-table-body-cell">
                {{ test.lecInstructorName }}
              </th>
              <th class="std-test-table-body-cell">
                {{ test.lecRoomName }}
              </th>
              <th class="std-test-table-body-cell">
                {{ test.testBeginDate?.substring(0, 10) }}
              </th>
              <th class="std-test-table-body-cell">
                {{ test.testEndDate?.substring(0, 10) }}
              </th>
              <th class="std-test-table-body-cell">
                <button
                  id="std-test-submit-detail-btn"
                  :disabled="disableTestBtn(test.testAvailable, test.submitYn)"
                  @click="openTestModal(test.lecId, test.testId, test.lecStudentId, false)"
                >
                  {{ testStatus(test.testAvailable, test.submitYn) }}
                </button>
              </th>
              <th class="std-test-table-body-cell">
                <button
                  id="std-test-score-detail-btn"
                  :disabled="disableTestResultBtn(test.testAvailable, test.scoreYn)"
                  @click="openTestModal(test.lecId, test.testId, test.lecStudentId, true)"
                >
                  {{ testResultStatus(test.testAvailable, test.scoreYn) }}
                </button>
              </th>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
  <PageNavigation :items-per-page="5" :total-items="listCount" :on-page-change="searchTest" />
  <TestModal
    v-if="modalState.type === 'testResult' && modalState.isOpen"
    :test-id="lecTestId"
    :lec-id="lectureId"
    :student-id="lecStudentId"
  />
</template>
<style>
@import './styled.css';
</style>
