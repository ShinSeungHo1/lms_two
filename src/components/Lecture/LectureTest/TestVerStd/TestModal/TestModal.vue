<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { ElMessage } from 'element-plus';
import { computed, onMounted, ref } from 'vue';

const props = defineProps({
  lecId: {
    type: Number,
    default: 0,
  },
  testId: {
    type: Number,
    default: 0,
  },
  studentId: {
    type: String,
    default: '',
  },
});

const modalState = useModalState();
const scoreVariable = ref({});
const questionVariable = ref({});
const questionList = ref([]);

const isToggle = ref(false);

const changeToggle = () => {
  isToggle.value = !isToggle.value;
};

const searchTestResultDetail = () => {
  const param = {
    testId: props.testId,
    lecId: props.lecId,
    studentId: props.studentId,
  };

  const urlParam = new URLSearchParams(param);

  if (modalState.payload) testResult(urlParam);
  else testExample(urlParam);
};
/**testEXample부터 할차례!!!!!!!!! */
const testExample = (urlParam) => {
  axios.post('/api/lecture/testQuestionNOptionInfoDetail.do', urlParam).then((res) => {
    console.log(res);
    const testQuestionInfoDetail = res.data.testQuestionInfoDetail;
    const testQuestionOptionInfoDetail = res.data.testQuestionOptionInfoDetail;
    const testSubmitOptionDetail = res.data.testSubmitOptionDetail;

    if (testQuestionInfoDetail.length === 0 && testQuestionOptionInfoDetail.length === 0) {
      ElMessage.error('문제가 아직 출제되지 않았습니다.');
      return;
    }

    const isSubmitted = Array.isArray(testSubmitOptionDetail) && testSubmitOptionDetail.length > 0;

    testQuestionInfoDetail.forEach((question) => {
      const questionId = question.questionId;
      const variable = {};
      variable.questionId = question.questionId;
      variable.questionNumber = question.questionNumber;
      variable.questionContent = question.questionContent;
      variable.questionScore = question.questionScore;

      const options = testQuestionOptionInfoDetail.filter((option) => {
        return option.questionId === questionId;
      });

      variable.options = [];
      options.forEach((option) => {
        const inputName = `question_${questionId}`;
        const inputId = `${inputName}_option_${option.optionId}`;
        const optionNumber = `보기 ${option.optionId}`;

        let isChecked = false;
        let isDisabled = false;

        if (isSubmitted) {
          const submitted = testSubmitOptionDetail.find((submit) => {
            return submit.questionId === questionId && submit.optionID === option.optionId;
          });
          if (submitted) {
            isChecked = true;
            isDisabled = true;
          }
        } else {
          const otherSubmitted = testSubmitOptionDetail.find((submit) => {
            return submit.questionId === questionId;
          });

          if (otherSubmitted) {
            isDisabled = true;
          }
        }

        option.inputName = inputName;
        option.inputId = inputId;
        option.isChecked = isChecked;
        option.isDisabled = isDisabled;
        option.optionNumber = optionNumber;

        variable.options.push(option);
      });
      questionVariable.value = variable;
      questionList.value.push(questionVariable.value);
    });
  });
};

const testResult = (urlParam) => {
  axios.post('/api/lecture/testTakeSubmitResultDetail.do', urlParam).then((res) => {
    console.log(res);
    const testQuestionInfoDetail = res.data.testQuestionInfoDetail;
    const testQuestionOptionInfoDetail = res.data.testQuestionOptionInfoDetail;
    const testQuestionAnswerInfoDetail = res.data.testQuestionAnswerInfoDetail;
    const testSubmitOptionDetail = res.data.testSubmitOptionDetailValue;
    const testResultInfoValue = res.data.testResultInfoValue;

    const testScore = testResultInfoValue.testScore || 0;
    const resultDate = testResultInfoValue.testResultRegDate.split('.')[0] || '정보 없음';
    const questionCount = testQuestionInfoDetail.length;

    let totalScore = 0;

    testQuestionInfoDetail.forEach((question) => (totalScore += question.questionScore || 0));

    console.log('총점 : ', totalScore);

    let correctAnswerCount = 0;
    testQuestionInfoDetail.forEach((question) => {
      const questionId = question.questionId;
      const correctAnswer = testQuestionAnswerInfoDetail.find((answer) => {
        return answer.questionId === questionId;
      });

      const correctOptionId = correctAnswer ? correctAnswer.correctOptionId : null;

      const submittedAnswer = testSubmitOptionDetail.find((submit) => {
        return submit.questionId === questionId;
      });

      const submittedOptionId = submittedAnswer ? submittedAnswer.optionId : null;

      if (correctOptionId === submittedOptionId) {
        correctAnswerCount++;
      }
    });
    const variable = {};
    variable.totalScore = totalScore;
    variable.correctAnswerCount = correctAnswerCount;
    variable.questionCount = questionCount;
    variable.testScore = testScore;
    variable.resultDate = resultDate;

    scoreVariable.value = variable;

    const isSubmitted = Array.isArray(testSubmitOptionDetail) && testSubmitOptionDetail.length > 0;

    testQuestionInfoDetail.forEach((question) => {
      const questionId = question.questionId;
      const variable = {};
      variable.questionId = question.questionId;
      variable.questionNumber = question.questionNumber;
      variable.questionContent = question.questionContent;
      variable.questionScore = question.questionScore;

      const options = testQuestionOptionInfoDetail.filter((option) => {
        return option.questionId === questionId;
      });

      const correctAnswer = testQuestionAnswerInfoDetail.find((answer) => {
        return answer.questionId === questionId;
      });

      const correctOptionId = correctAnswer ? correctAnswer.correctOptionId : null;

      const submittedAnswer = testSubmitOptionDetail.find((submit) => {
        return submit.questionId === questionId;
      });

      const submittedOptionId = submittedAnswer ? submittedAnswer.optionId : null;
      variable.options = [];
      options.forEach((option) => {
        const inputName = `question_${questionId}`;
        const inputId = `${inputName}_option_${option.optionId}`;
        const optionNumber = `보기 ${option.optionId}`;

        let isChecked = false;
        let isDisabled = false;

        if (isSubmitted) {
          isChecked = submittedOptionId === option.optionId;
          isDisabled = true;
        } else {
          const otherSubmitted = testSubmitOptionDetail.find((submit) => {
            return submit.questionId === questionId;
          });

          if (otherSubmitted) {
            isDisabled = true;
          }
        }

        const isCorrect = submittedOptionId === correctOptionId;
        let correctBackGround = null;
        if (isChecked) {
          if (isCorrect) {
            correctBackGround = true;
          } else {
            correctBackGround = false;
          }
        }

        const backGround = computed(() => {
          if (correctBackGround) {
            return {
              backgroundColor: '#d4edda',
              color: 'green',
            };
          } else if (correctBackGround === null) {
            return {};
          } else {
            return {
              backgroundColor: '#f8d7da',
              color: 'red',
            };
          }
        });

        let correctAnswer = '';
        if (option.optionId === correctOptionId) {
          correctAnswer = 'correct';
        } else if (isChecked) {
          correctAnswer = 'uncorrect';
        }

        option.backGround = backGround;
        option.correctBackGround = correctBackGround;
        option.correctAnswer = correctAnswer;
        option.inputName = inputName;
        option.inputId = inputId;
        option.isChecked = isChecked;
        option.isCorrect = isCorrect;
        option.isDisabled = isDisabled;
        option.optionNumber = optionNumber;

        variable.options.push(option);
      });
      questionVariable.value = variable;
      questionList.value.push(questionVariable.value);
    });
  });
};

onMounted(() => {
  searchTestResultDetail();
});
</script>
<template>
  <div class="test-result-info-container">
    <div v-if="modalState.payload" class="test-result-info-content">
      <div class="test-result-head-label">시험 결과</div>
      <dl class="test-result-info-list">
        <dt>채점 문제수/채점 점수</dt>
        <dd>
          {{
            `${scoreVariable.correctAnswerCount}/${scoreVariable.questionCount}(${scoreVariable.testScore}/${scoreVariable.totalScore}점)`
          }}
        </dd>

        <dt>전체 문제 수</dt>
        <dd>{{ scoreVariable.questionCount }}</dd>

        <dt>시험 제출일</dt>
        <dd>{{ scoreVariable.resultDate }}</dd>
      </dl>
    </div>
    <div
      v-for="(question, index) in questionList"
      :key="`q-${index}`"
      class="test-question-collect-container"
    >
      <div class="test-question-collect-label" @click="changeToggle">
        {{
          `Q${question.questionNumber}. ${question.questionContent} (${question.questionScore}점)`
        }}
      </div>
      <div v-if="isToggle" class="test-question-example-container">
        <fieldset class="test-qeustion-example-options">
          <div v-for="(option, idx) in question.options" :key="`opt-${idx}`">
            <input
              :id="option.inputId"
              :name="option.inputName"
              :value="option.optionId"
              :checked="option.isChecked"
              :disabled="option.isDisabled"
              type="radio"
            />
            <label :for="option.inputId" :style="option.backGround"
              >{{ `${option.optionNumber}. ${option.optionContent}` }}
              <span v-if="option.correctAnswer === 'correct'" style="color: green">(정답)</span>
              <span v-if="option.correctAnswer === 'uncorrect'" style="color: red">(오답)</span>
            </label>
          </div>
        </fieldset>
      </div>
    </div>
  </div>
</template>
<style>
@import './styled.css';
</style>
