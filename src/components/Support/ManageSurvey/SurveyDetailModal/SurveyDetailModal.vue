<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref } from 'vue';

const modalState = useModalState();

const props = defineProps({
  lecId: {
    type: Number,
    default: 0,
  },
  loginId: {
    type: String,
    default: '',
  },
});

const detailList = ref([]);
const averageLabel = ref('');

const getSurveyDetail = () => {
  const param = {
    lecId: props.lecId,
    loginId: props.loginId,
  };

  const urlParam = new URLSearchParams(param);

  axios.post('/api/support/getSurveyDetail.do', urlParam).then((res) => {
    console.log(`${param.loginId}님의 ${param.lecId}강의 설문조사 상세사항 불러오기 완료`);
    detailList.value = res.data.detail;
    let sum = 0;
    let avg = 0;
    let avgRounded;
    let avgLabel;

    const count = detailList.value.length;

    const labels = ['평가 없음', '매우 나쁨', '나쁨', '보통', '좋음', '매우 좋음'];

    detailList.value.forEach((item) => {
      sum += item.surveyResult;
      const label = labels[item.surveyResult] || item.surveyResult;
      item.label = label;
    });

    avg = sum / count;
    avgRounded = Math.round(avg);
    avgLabel = labels[avgRounded] || avgRounded.toFixed(1);
    averageLabel.value = avgLabel;
    console.log(detailList.value);
  });
};

const closeModal = () => {
  modalState.$patch({ isOpen: false });
};

onMounted(() => {
  getSurveyDetail();
});
</script>
<template>
  <div class="modal-overlay" @click="closeModal">
    <div class="modal-container" @click.stop>
      <!-- 헤더 -->
      <div class="modal-header">
        <h2>상세내역</h2>
        <button class="close-btn" @click="closeModal">×</button>
      </div>

      <!-- 컨텐츠 -->
      <div class="modal-content">
        <div v-for="detail in detailList" :key="detail.surveyId" class="content-row">
          <div class="question-cell">{{ detail.surveyContent }}</div>
          <div class="answer-cell">{{ detail.label }}</div>
        </div>
        <div class="content-row">
          <div class="question-cell">평균</div>
          <div class="answer-cell">{{ averageLabel }}</div>
        </div>
      </div>

      <!-- 버튼 -->
      <div class="modal-footer">
        <button class="confirm-btn" @click="closeModal">닫기</button>
      </div>
    </div>
  </div>
</template>

<style>
@import './styled.css';
</style>
