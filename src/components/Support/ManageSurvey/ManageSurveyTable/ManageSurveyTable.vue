<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import SurveyDetailModal from '../SurveyDetailModal/SurveyDetailModal.vue';
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { computed, ref, watch } from 'vue';

const modalState = useModalState();

const props = defineProps({
  manageItem: {
    type: String,
    default: '',
  },
});

const chooseHeader = () => {
  if (props.manageItem === 'completedSurvey') {
    return ['No.', '과목명', '학생ID', '학생명', '상세보기'];
  } else {
    return ['No.', '과목명', '강사 이름', '평균', '응답인원', '완료율'];
  }
};

const headerList = computed(() => chooseHeader());
const loadList = ref([]);
const resultList = ref([]);
const resultCnt = ref(0);
const labels = ['평가 없음', '매우 나쁨', '나쁨', '보통', '좋음', '매우 좋음'];

const loginId = ref('');
const lecId = ref(0);

const isBold = ref(false);

const loadResultList = (cPage = 1) => {
  // page = page || 1;
  const param = {
    currentPage: cPage,
    pageSize: 10,
  };

  const urlParam = new URLSearchParams(param);

  axios.post('/api/support/getResultList.do', urlParam).then((res) => {
    console.log('설문 결과 조회 성공');
    loadList.value = res.data.fixedRes;
    resultCnt.value = res.data.resultCnt;
    const start = (param.currentPage - 1) * param.pageSize;
    const end = start + param.pageSize;
    const pageItems = loadList.value.slice(start, end);

    pageItems.forEach((item, idx) => {
      const index = item.avgScore;
      const label = labels[index];
      const rowNum = start + idx + 1;
      const respondentHtml = item.respondentCount + '명';
      const completionHtml = item.completionRate + '%';

      if (index > 0) {
        isBold.value = true;
      }

      item.label = label;
      item.rowNum = rowNum;
      item.respondentHtml = respondentHtml;
      item.completionHtml = completionHtml;
    });

    loadList.value = pageItems;
  });
};

const loadCompletedList = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 10,
  };

  const urlParam = new URLSearchParams(param);

  axios.post('/api/support/getCompletedList.do', urlParam).then((res) => {
    console.log('완료 목록 조회 불러오기 성공!');
    resultList.value = res.data.resultList;
    resultCnt.value = res.data.resultCnt;
    const pageItems = resultList.value;
    pageItems.forEach((item, index) => {
      const rowNum = (res.data.currentPage - 1) * res.data.pageSize + (index + 1);
      item.rowNum = rowNum;
    });

    resultList.value = pageItems;
    console.log(resultList.value);
  });
};

const showModal = (login, lec) => {
  modalState.$patch({ isOpen: true, type: 'detailSurvey' });
  loginId.value = login;
  lecId.value = lec;
};

watch(
  () => props.manageItem,
  () => {
    if (props.manageItem === 'getResult') loadResultList();
    if (props.manageItem === 'completedSurvey') loadCompletedList();
  },
);
</script>

<template>
  <div class="table-container">
    <!-- 완료 목록 조회테이블 -->
    <table v-if="props.manageItem === 'completedSurvey'" class="data-table">
      <thead class="table-header">
        <tr>
          <th
            v-for="(header, index) in headerList"
            :key="index"
            :style="{ width: header.width }"
            class="table-header-cell"
          >
            {{ header }}
          </th>
        </tr>
      </thead>
      <tbody class="table-body">
        <!-- 데이터가 있는 경우 -->
        <tr v-for="(item, idx) in resultList" :key="idx" class="table-row">
          <td class="table-cell">{{ item.rowNum }}</td>
          <td class="course-name table-cell">{{ item.lecName }}</td>
          <td class="table-cell">{{ item.loginId }}</td>
          <td class="table-cell">{{ item.stName }}</td>
          <td class="table-cell">
            <button @click="showModal(item.loginId, item.lecId)">보기</button>
          </td>
        </tr>

        <!-- 데이터가 없는 경우 -->
        <tr v-if="resultList.length === 0" class="empty-row">
          <td :colspan="headerList.length" class="empty-cell">데이터가 없습니다.</td>
        </tr>
      </tbody>
    </table>
    <!-- 결과 조회 테이블 -->
    <table v-if="props.manageItem === 'getResult'" class="data-table">
      <thead class="table-header">
        <tr>
          <th
            v-for="(header, index) in headerList"
            :key="index"
            :style="{ width: header.width }"
            class="table-header-cell"
          >
            {{ header }}
          </th>
        </tr>
      </thead>
      <tbody class="table-body">
        <!-- 데이터가 있는 경우 -->
        <tr v-for="(item, idx) in loadList" :key="idx" class="table-row">
          <td class="table-cell" :class="{ strong: isBold }">{{ item.rowNum }}</td>
          <td class="course-name table-cell" :class="{ strong: isBold }">{{ item.lecName }}</td>
          <td class="table-cell" :class="{ strong: isBold }">{{ item.lecInstructorName }}</td>
          <td class="table-cell" :class="{ strong: isBold }">{{ item.label }}</td>
          <td class="table-cell" :class="{ strong: isBold }">{{ item.respondentHtml }}</td>
          <td class="table-cell">
            <div class="progress-container">
              <div class="progress-bar" :style="{ width: item.completionRate + '%' }"></div>
              <span class="progress-text">{{ item.completionHtml }}</span>
            </div>
          </td>
        </tr>

        <!-- 데이터가 없는 경우 -->
        <tr v-if="loadList.length === 0" class="empty-row">
          <td :colspan="headerList.length" class="empty-cell">데이터가 없습니다.</td>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation
    v-if="props.manageItem === 'getResult'"
    :items-per-page="10"
    :total-items="resultCnt"
    :on-page-change="loadResultList"
  />
  <PageNavigation
    v-if="props.manageItem === 'completedSurvey'"
    :items-per-page="10"
    :total-items="resultCnt"
    :on-page-change="loadCompletedList"
  />
  <SurveyDetailModal
    v-if="modalState.type === 'detailSurvey' && modalState.isOpen"
    :lec-id="lecId"
    :login-id="loginId"
  />
</template>

<style>
@import './styled.css';
</style>
