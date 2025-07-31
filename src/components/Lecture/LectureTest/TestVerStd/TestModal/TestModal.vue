<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { ElMessage, ElMessageBox } from 'element-plus';
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

const emits = defineEmits(['success']);

const modalState = useModalState();
const scoreVariable = ref({});
const questionVariable = ref({});
const questionList = ref([]);

const testQuestionInfoDetail = ref([]);
const testQuestionOptionInfoDetail = ref([]);
const testQuestionAnswerInfoDetail = ref([]);
const testSubmitOptionDetail = ref([]);
const testResultInfoValue = ref({});

const expandedQuestions = ref([]);
const showBtnSubmitTest = ref(false);
const isDisabledRadio = ref(false);

const emitsAndModalClose = (emit) => {
  emits(emit);
  modalState.$patch({ isOpen: false });
};

const toggleExpandedQuestions = (index) => {
  const expandedIndex = expandedQuestions.value.indexOf(index);
  if (expandedIndex > -1) {
    // 이미 열려있으면 → 닫기
    expandedQuestions.value.splice(expandedIndex, 1);
  } else {
    // 닫혀있으면 → 열기
    expandedQuestions.value.push(index);
  }
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
    testQuestionInfoDetail.value = res.data.testQuestionInfoDetail;
    testQuestionOptionInfoDetail.value = res.data.testQuestionOptionInfoDetail;
    testSubmitOptionDetail.value = res.data.testSubmitOptionDetail;

    if (
      testQuestionInfoDetail.value.length === 0 &&
      testQuestionOptionInfoDetail.value.length === 0
    ) {
      ElMessage.error('문제가 아직 출제되지 않았습니다.');
      return;
    }

    const isSubmitted =
      Array.isArray(testSubmitOptionDetail.value) && testSubmitOptionDetail.value.length > 0;

    testQuestionInfoDetail.value.forEach((question) => {
      const questionId = question.questionId;
      const variable = {};
      variable.questionId = question.questionId;
      variable.questionNumber = question.questionNumber;
      variable.questionContent = question.questionContent;
      variable.questionScore = question.questionScore;

      const options = testQuestionOptionInfoDetail.value.filter((option) => {
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
          const submitted = testSubmitOptionDetail.value.find((submit) => {
            return submit.questionId === questionId && submit.optionId === option.optionId;
          });
          console.log('disabled false');
          if (submitted) {
            isChecked = true;
            isDisabled = true;
          } else {
            const otherSubmitted = testSubmitOptionDetail.value.find((submit) => {
              return submit.questionId === questionId;
            });

            if (otherSubmitted) {
              isDisabled = true;
            }
          }
        }
        showBtnSubmitTest.value = true;

        if (isDisabled) {
          isDisabledRadio.value = true;
        }
        console.log(isDisabledRadio.value);
        option.isDisabled = isDisabled;
        option.inputName = inputName;
        option.inputId = inputId;
        option.isChecked = isChecked;
        option.optionNumber = optionNumber;
        variable.options.push(option);
      });

      questionVariable.value = variable;
      questionList.value.push(questionVariable.value);
      console.log(questionList.value);
    });
  });
};

const testResult = (urlParam) => {
  axios.post('/api/lecture/testTakeSubmitResultDetail.do', urlParam).then((res) => {
    console.log(res);
    testQuestionInfoDetail.value = res.data.testQuestionInfoDetail;
    testQuestionOptionInfoDetail.value = res.data.testQuestionOptionInfoDetail;
    testQuestionAnswerInfoDetail.value = res.data.testQuestionAnswerInfoDetail;
    testSubmitOptionDetail.value = res.data.testSubmitOptionDetailValue;
    testResultInfoValue.value = res.data.testResultInfoValue;

    const testScore = testResultInfoValue.value.testScore || 0;
    const resultDate = testResultInfoValue.value.testResultRegDate.split('.')[0] || '정보 없음';
    const questionCount = testQuestionInfoDetail.value.length;

    let totalScore = 0;

    testQuestionInfoDetail.value.forEach((question) => (totalScore += question.questionScore || 0));

    console.log('총점 : ', totalScore);

    let correctAnswerCount = 0;
    testQuestionInfoDetail.value.forEach((question) => {
      const questionId = question.questionId;
      const correctAnswer = testQuestionAnswerInfoDetail.value.find((answer) => {
        return answer.questionId === questionId;
      });

      const correctOptionId = correctAnswer ? correctAnswer.correctOptionId : null;

      const submittedAnswer = testSubmitOptionDetail.value.find((submit) => {
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

    const isSubmitted =
      Array.isArray(testSubmitOptionDetail.value) && testSubmitOptionDetail.value.length > 0;

    testQuestionInfoDetail.value.forEach((question) => {
      const questionId = question.questionId;
      const variable = {};
      variable.questionId = question.questionId;
      variable.questionNumber = question.questionNumber;
      variable.questionContent = question.questionContent;
      variable.questionScore = question.questionScore;

      const options = testQuestionOptionInfoDetail.value.filter((option) => {
        return option.questionId === questionId;
      });

      const correctAnswer = testQuestionAnswerInfoDetail.value.find((answer) => {
        return answer.questionId === questionId;
      });

      const correctOptionId = correctAnswer ? correctAnswer.correctOptionId : null;

      const submittedAnswer = testSubmitOptionDetail.value.find((submit) => {
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
          const otherSubmitted = testSubmitOptionDetail.value.find((submit) => {
            return submit.questionId === questionId;
          });

          if (otherSubmitted) {
            isDisabled = true;
          }
        }

        if (isDisabled) {
          isDisabledRadio.value = true;
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

const submitTest = () => {
  const answers = [];
  const unAnsweredNumbers = [];
  const submitDate = getCurrentDateTime();
  const hasUnAnswered = ref(false);

  questionList.value.forEach((question, index) => {
    const questionNumber = question.questionNumber || index + 1;

    const selectedOption = question.options.find((option) => option.isChecked);

    if (!selectedOption) {
      hasUnAnswered.value = true;
      unAnsweredNumbers.push(`${questionNumber}번`);
    } else {
      console.log(selectedOption);
      answers.push({
        questionId: question.questionId,
        testId: selectedOption.testId,
        lecId: selectedOption.lecId,
        optionId: selectedOption.optionId,
        studentId: props.studentId,
        submitDate: submitDate,
      });
    }
  });

  console.log(answers);
  if (unAnsweredNumbers.length > 0) {
    ElMessage.error(`다음 문항에 답변하지 않았습니다:\n• ${unAnsweredNumbers.join(',')}`);
    return;
  }

  ElMessageBox.confirm('정말로 시험을 제출하시겠습니까?', {
    confirmButtonText: '네!!',
    cancelButtonText: '아니요!!',
    type: 'warning',
  })
    .then(() => {
      const param = {
        answers: answers,
        testId: props.testId,
        lecId: props.lecId,
        studentId: props.studentId,
        submitDate: submitDate,
      };

      axios
        .post('/api/lecture/testTakeSubmit.do', param)
        .then((res) => {
          if (res.data.result === 'success') {
            ElMessage.success('시험이 성공적으로 제출되었습니다.');
            emitsAndModalClose('success');
          } else if (res.data.result === 'isExist') {
            ElMessage.error('이미 제출된 시험입니다.');
            emitsAndModalClose('success');
          } else {
            ElMessage.error('다시 시도해 주세요');
          }
        })
        .catch((err) => {
          ElMessage.error('서버 오류 발생');
          console.log('시험 제출 서버 오류 발생 err : ', err);
        });
    })
    .catch(() => {
      return;
    });
};

const pickAnswer = (questionId, optionId) => {
  questionList.value.forEach((question) => {
    if (question.questionId === questionId) {
      question.options.forEach((option) => {
        option.isChecked = option.optionId === optionId;
      });
    }
  });
};

const getCurrentDateTime = () => {
  const now = new Date();
  const yyyy = now.getFullYear();
  const mm = ('0' + (now.getMonth() + 1)).slice(-2);
  const dd = ('0' + now.getDate()).slice(-2);
  const hh = ('0' + now.getHours()).slice(-2);
  const min = ('0' + now.getMinutes()).slice(-2);
  return `${yyyy}-${mm}-${dd} ${hh}:${min}:00`;
};

const exitModal = () => {
  modalState.$patch({ isOpen: false });
};

onMounted(() => {
  searchTestResultDetail();
});
</script>
<template>
  <div class="test-modal-backdrop">
    <div class="test-modal-content">
      <div class="test-modal-title-container">
        <div class="test-modal-title">시험 응시</div>
        <button class="clost-test-modal-btn" @click="exitModal">x</button>
      </div>
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
          <div class="test-question-collect-label" @click="toggleExpandedQuestions(index)">
            {{
              `Q${question.questionNumber}. ${question.questionContent} (${question.questionScore}점)`
            }}
          </div>
          <div v-if="expandedQuestions.includes(index)" class="test-question-example-container">
            <fieldset class="test-qeustion-example-options">
              <div v-for="(option, idx) in question.options" :key="`opt-${idx}`">
                <input
                  :id="option.inputId"
                  :name="option.inputName"
                  :value="option.optionId"
                  :checked="option.isChecked"
                  :disabled="isDisabledRadio"
                  type="radio"
                  @change="pickAnswer(question.questionId, option.optionId)"
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
        <div class="test-btn-container">
          <button v-if="showBtnSubmitTest" class="test-submit-container" @click="submitTest">
            제출
          </button>
          <button class="test-cancel-btn" @click="exitModal">취소</button>
        </div>
      </div>
    </div>
  </div>
</template>
<style>
@import './styled.css';
</style>
