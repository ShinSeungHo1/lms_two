<script setup>
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { ElMessage } from 'element-plus';

const questionList = ref([]);
const answerList = ref([]);
const currentIdx = ref(0);

const options = ref([]);
const selectedOption = ref(0);

const props = defineProps({
  lecId: {
    type: Number,
    default: 0,
  },
});

const controllerCondition = (
  defaultSurveyPageBool,
  haveQuestionsBool,
  disablePrevBtnBool,
  disableNextBtnBool,
) => {
  defaultSurveyPage.value = defaultSurveyPageBool;
  haveQuestions.value = haveQuestionsBool;
  disablePrevBtn.value = disablePrevBtnBool;
  disableNextBtn.value = disableNextBtnBool;
};

const createOptions = () => {
  options.value = [];
  selectedOption.value = 0;
  options.value.push({
    index: 1,
    text: '매우나쁨',
  });
  options.value.push({
    index: 2,
    text: '나쁨',
  });
  options.value.push({
    index: 3,
    text: '보통',
  });
  options.value.push({
    index: 4,
    text: '좋음',
  });
  options.value.push({
    index: 5,
    text: '매우좋음',
  });
};

const defaultSurveyPage = ref(true);
const haveQuestions = ref(false);
const disablePrevBtn = ref(true);
const disableNextBtn = ref(true);
const isFinished = ref(false);

const initSurvey = () => {
  console.log('가져온 prop', props.lecId);
  if (props.lecId === 0) {
    controllerCondition(true, false, true, true);
    return;
  }

  const param = {
    lecId: props.lecId,
  };

  const urlParam = new URLSearchParams(param);

  console.log('Param 값 : ', param);
  axios.post('/api/support/getSurveyContents.do', urlParam).then((res) => {
    console.log('질문 사항 불러오기 성공', res);
    if (res.data.length === 0) {
      controllerCondition(false, false, true, true);
      return;
    }
    controllerCondition(false, true, currentIdx.value === 0, false);
    questionList.value = res.data;
  });
};
/** idx관련 로직은 여기서 처리하는 게 맞는듯? */

const nextSurvey = () => {
  if (currentIdx.value === questionList.value.length - 1) {
    if (answerList.value.includes(0)) {
      ElMessage.error('모든 질문에 답해주세요.');
      return;
    }
    answerList.value.push(selectedOption.value);
    finishSurvey();
  } else {
    answerList.value.push(selectedOption.value);
    currentIdx.value++;
  }
};

const prevSurvey = () => {
  if (currentIdx.value > 0) {
    currentIdx.value--;
  }
};

const finishSurvey = () => {
  for (let i = 0; i < answerList.value.length; i++) {
    saveResult(i + 1, answerList.value[i]);
  }
  ElMessage.success('설문이 완료되었습니다.');
  isFinished.value = true;
  disablePrevBtn.value = true;
  disableNextBtn.value = true;
};

const saveResult = (surveyId, surveyResult) => {
  const param = {
    surveyId: surveyId,
    surveyResult: surveyResult,
    lecId: props.lecId,
  };

  const urlParam = new URLSearchParams(param);

  axios.post('/api/support/saveResult.do', urlParam).then((res) => {
    if (res.data.result === 'success' && surveyId === answerList.value.length) {
      ElMessage.success('저장 성공!');
    } else if (res.data.result === 'fail') {
      ElMessage.error('저장 실패!');
    }
  });
};

watch([() => currentIdx.value, () => props.lecId], () => {
  initSurvey();
  createOptions();
});

onMounted(() => {
  initSurvey();
  createOptions();
});
</script>

<template>
  <div class="survey-container">
    <div class="survey-header">
      <h2 class="survey-title">설문조사</h2>
    </div>

    <div class="survey-content">
      <!-- Q1 질문 -->
      <div
        v-if="haveQuestions === false && defaultSurveyPage === true && isFinished === false"
        class="question-section"
      >
        <h3 class="question-title">먼저 강의를 선택해주세요.</h3>
      </div>
      <div
        v-if="haveQuestions === false && defaultSurveyPage === false && isFinished === false"
        class="question-section"
      >
        <h3 class="question-title">이미 완료한 설문입니다.</h3>
      </div>
      <div v-if="isFinished === true" class="question-section">
        <h3 class="question-title">수고하셨습니다.</h3>
      </div>
      <div
        v-if="haveQuestions === true && defaultSurveyPage === false && isFinished === false"
        class="question-section"
      >
        <h3 class="question-title">{{ questionList[currentIdx] }}</h3>

        <div class="radio-group">
          <div v-for="option in options" :key="option.index" class="radio-item">
            <input
              v-model="selectedOption"
              :value="option.index"
              class="radio-input"
              type="radio"
            />
            <label class="radio-label">
              {{ option.text }}
            </label>
          </div>
        </div>
      </div>

      <!-- 버튼 영역 -->
      <div class="button-section">
        <button
          type="button"
          class="btn btn-secondary"
          :disabled="disablePrevBtn"
          @click="prevSurvey"
        >
          이전
        </button>
        <button
          type="button"
          class="btn btn-primary"
          :disabled="disableNextBtn"
          @click="nextSurvey"
        >
          {{ currentIdx === questionList.length - 1 ? '완료' : '다음' }}
        </button>
      </div>
    </div>
  </div>
</template>

<style>
@import './styled.css';
</style>
