<script setup>
import { onMounted, onUnmounted, ref } from 'vue';
import { useUserInfo } from '@/stores/loginInfoState';
import axios from 'axios';
import { ElMessage } from 'element-plus';
import { useModalState } from '@/stores/modalState';

const { user } = useUserInfo();

const props = defineProps({
  qnaId: {
    type: Number,
    default: 0,
  },
});

const emits = defineEmits(['deleteSuccess', 'saveSuccess', 'updateSuccess', 'answerFail']);

const modalState = useModalState();

const lecList = ref([]);
const filteredLecList = ref([]);
const searchKeyword = ref('');
const qnaTitle = ref('');
const qnaContent = ref('');
const lecId = ref(0);
const lecName = ref('');
const qnaAnswer = ref('');
const qnaAnswerDate = ref('');
const qnaRegDate = ref('');
const qnaAnswerWriter = ref('');

const showQuestionSave = ref(false);
const showQuestionUpdate = ref(false);
const showQuestionDelete = ref(false);
const showAnswerSave = ref(false);
const showAnswerUpdate = ref(false);
const showAnswerDelete = ref(false);
const isReadonlyAnswer = ref(false);

const isDropdownOpen = ref(false);

const refQnaTitle = ref();
const refQnaContent = ref();
const refLecName = ref();
const dropdownRef = ref();

const modalEmitFunc = (emit) => {
  modalState.$patch({ isOpen: false });
  emits(emit);
};

const searchLecList = () => {
  const param = {
    loginId: user.loginId,
  };

  const urlParam = new URLSearchParams(param);
  axios
    .post('/api/support/getQnaLectureListBody.do', urlParam)
    .then((res) => {
      console.log('강의 목록 불러오기 성공: ' + res.data);
      lecList.value = res.data.lectureList;
    })
    .catch((err) => {
      console.log('강의 목록 불러오기 실패...: ' + err);
    });
};

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
  if (isDropdownOpen.value) {
    searchKeyword.value = '';
    filteredLecList.value = lecList.value;
  }
};

const onInputSearch = () => {
  isDropdownOpen.value = true;
  const keyword = searchKeyword.value.toLowerCase();
  if (keyword === '') {
    filteredLecList.value = lecList.value;
  } else {
    filteredLecList.value = lecList.value.filter((lecture) =>
      lecture.lecName.toLowerCase().includes(keyword),
    );
  }
};

const onInputFocus = () => {
  isDropdownOpen.value = true;
  searchKeyword.value = '';
  filteredLecList.value = lecList.value;
};

const closeDropdown = (event) => {
  if (dropdownRef.value && !dropdownRef.value.contains(event.target)) {
    isDropdownOpen.value = false;
  }
};

const selectLecture = (lecture) => {
  lecId.value = lecture.lecId;
  lecName.value = lecture.lecName;
  searchKeyword.value = lecture.lecName;
  isDropdownOpen.value = false;
};

const qnaDetail = () => {
  const param = {
    qnaId: props.qnaId,
  };

  const urlParam = new URLSearchParams(param);
  axios
    .post('/api/support/qnaDetail.do', urlParam)
    .then((res) => {
      console.log(`!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!`);
      console.log(res);
      qnaTitle.value = res.data.qnaTitle;
      qnaContent.value = res.data.qnaContent;
      qnaAnswer.value = res.data.qnaAnswer;
      lecId.value = res.data.lecId;
      lecName.value = res.data.lecName;
      qnaAnswerDate.value = res.data.qnaAnswerDate;
      qnaRegDate.value = res.data.qnaRegDate;
      qnaAnswerWriter.value = user.loginId;

      const userType = user.userType;

      if (modalState.type === 'update') {
        if (qnaAnswer.value && qnaTitle.value !== '') {
          showAnswerUpdate.value = true;
          showAnswerSave.value = false;
          isReadonlyAnswer.value = false;
        } else {
          showAnswerUpdate.value = false;
          showAnswerSave.value = true;
        }

        if (userType === 'S') {
          showQuestionUpdate.value = true;
          showQuestionDelete.value = true;
          showAnswerUpdate.value = false;
          isReadonlyAnswer.value = true;
        }
        if (userType === 'T') {
          showAnswerDelete.value = true;
        }
        if (userType === 'M') {
          showQuestionDelete.value = true;
          showAnswerDelete.value = true;
          showAnswerUpdate.value = false;
          isReadonlyAnswer.value = true;
        }
      } else {
        if (userType === 'S') showQuestionSave.value = true;
      }

      console.log('QnA detail 불러오기 성공!');
    })
    .catch((err) => {
      console.log('QnA detail 불러오기 실패 : ' + err);
    });
};

const saveQuestion = () => {
  const param = {
    qnaId: props.qnaId,
    qnaTitle: qnaTitle.value,
    qnaContent: qnaContent.value,
    lecId: lecId.value,
  };

  if (!param.qnaTitle || param.qnaTitle === '') {
    ElMessage.error('제목을 입력하여주세요');
    refQnaTitle.value.focus();
    return;
  }

  if (!param.qnaContent || param.qnaContent === '') {
    ElMessage.error('내용을 입력해주세요');
    refQnaContent.value.focus();
    return;
  }

  if (!param.lecId) {
    ElMessage.error('강의를 선택해주세요');
    if (dropdownRef.value) {
      dropdownRef.value.click();
    }
    return;
  }

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/support/saveQuestion.do', urlParam)
    .then((res) => {
      console.log('Question 저장 성공하였습니다' + res);
      ElMessage.success('저장 성공!');
      modalEmitFunc('saveSuccess');
    })
    .catch((err) => {
      ElMessage.error('저장 실패!');
      console.log('저장에 실패하였습니다...' + err);
    });
};

const updateQuestion = () => {
  const param = {
    qnaId: props.qnaId,
    qnaTitle: qnaTitle.value,
    qnaContent: qnaContent.value,
    lecId: lecId.value,
  };

  if (!param.qnaTitle || param.qnaTitle === '') {
    ElMessage.error('제목을 입력하여주세요');
    refQnaTitle.value.focus();
    return;
  }

  if (!param.qnaContent || param.qnaContent === '') {
    ElMessage.error('내용을 입력해주세요');
    refQnaContent.value.focus();
    return;
  }

  if (!param.lecId) {
    ElMessage.error('강의를 선택해주세요');
    if (dropdownRef.value) {
      dropdownRef.value.click();
    }
    return;
  }

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/support/updateQuestion.do', urlParam)
    .then((res) => {
      ElMessage.success('수정되었습니다!!!!');
      modalEmitFunc('updateSuccess');
      console.log('질문 update 성공! : ' + res.data);
    })
    .catch((err) => {
      ElMessage.error('수정에 실패했습니다!');
      console.log('질문 update 실패! : ' + err.data);
    });
};

const saveAnswer = () => {
  const param = {
    questionId: props.qnaId,
    answerContent: qnaAnswer.value,
  };

  const urlParam = new URLSearchParams(param);
  axios.post('/api/support/saveAnswer.do', urlParam).then((res) => {
    if (res.data.result === 'success') {
      ElMessage.success('저장 성공!');
      console.log('답변 save 성공!');
      modalEmitFunc('saveSuccess');
    } else {
      ElMessage.error('해당 강의의 강사만 답변할 수 있습니다.');
      console.log('답변 save 실패!');
      modalEmitFunc('saveFail');
    }
  });
};

const updateAnswer = () => {
  const param = {
    questionId: props.qnaId,
    answerContet: qnaAnswer.value,
  };

  const urlParam = new URLSearchParams(param);
  axios.post('/api/support/updateAnswer.do', urlParam).then((res) => {
    if (res.data.result === 'success') {
      ElMessage.success('답변 수정되었습니다!!');
      console.log('답변 update 성공');
      modalEmitFunc('updateSuccess');
    } else {
      ElMessage.error('해당 강의의 강사만 수정할 수 있습니다.');
      console.log('답변 update 실패');
      modalEmitFunc('answerFail');
    }
  });
};

const deleteQuestion = () => {
  const param = {
    questionId: props.qnaId,
  };

  const urlParam = new URLSearchParams(param);
  axios
    .post('/api/support/deleteQuestion.do', urlParam)
    .then((res) => {
      console.log('삭제 성공 여부: ' + res);
      ElMessage.success('질문 사항 삭제 완료');
      emits('deleteSuccess');
      modalState.$patch({ isOpen: false });
    })
    .catch((err) => {
      ElMessage.error('삭제 실패..');
      console.log('삭제 실패 여부: ' + err);
    });
};

const deleteAnswer = () => {
  const param = {
    questionId: props.qnaId,
  };

  const urlParam = new URLSearchParams(param);
  axios
    .post('/api/support/deleteAnswer.do', urlParam)
    .then((res) => {
      console.log('삭제 성공 여부: ' + res);
      ElMessage.success('대답 사항 삭제 완료');
      emits('deleteSuccess');
      modalState.$patch({ isOpen: false });
    })
    .catch((err) => {
      ElMessage.error('삭제 실패....');
      console.log('삭제 실패 여부: ' + err);
    });
};

onMounted(() => {
  console.log(user);
  qnaDetail();
  searchLecList();
  document.addEventListener('click', closeDropdown);
});

onUnmounted(() => {
  document.removeEventListener('click', closeDropdown);
});
</script>

<template>
  <Teleport to="body">
    <div class="modal-overlay">
      <div class="modal-container">
        <!-- 모달 헤더 -->
        <div class="modal-header">
          <h2 class="modal-title">Q&A</h2>
          <button type="button" class="close-button" @click="modalState.$patch({ isOpen: false })">
            ×
          </button>
        </div>

        <!-- Q&A 폼 -->
        <div class="modal-content">
          <!-- 질문 섹션 -->
          <div class="question-section">
            <table class="form-table">
              <tbody>
                <tr>
                  <th class="form-label">강의명 <span class="required">*</span></th>
                  <td class="form-input">
                    <div ref="dropdownRef" class="lecture-dropdown-container">
                      <input
                        ref="refLecName"
                        v-model="lecName"
                        type="text"
                        class="input-field dropdown-input"
                        placeholder="클릭해서 강의 선택"
                        readonly
                        @click="toggleDropdown"
                        @input="onInputSearch"
                        @focus="onInputFocus"
                      />

                      <!-- 강의 목록 드롭다운 -->
                      <div v-if="isDropdownOpen" class="lecture-dropdown-list">
                        <div
                          v-for="lecture in lecList"
                          :key="lecture.lecId"
                          class="lecture-dropdown-item"
                          @click="selectLecture(lecture)"
                        >
                          {{ lecture.lecName }}
                        </div>
                        <div v-if="lecList.length === 0" class="lecture-dropdown-empty">
                          수강중인 강의가 없습니다.
                        </div>
                      </div>
                    </div>
                  </td>
                </tr>
                <tr>
                  <th class="form-label">제목 <span class="required">*</span></th>
                  <td class="form-input">
                    <input
                      ref="refQnaTitle"
                      v-model="qnaTitle"
                      type="text"
                      class="input-field"
                      placeholder="제목을 입력하세요"
                    />
                  </td>
                </tr>
                <tr>
                  <th class="form-label">내용 <span class="required">*</span></th>
                  <td class="form-input">
                    <textarea
                      ref="refQnaContent"
                      v-model="qnaContent"
                      rows="4"
                      class="textarea-field"
                      placeholder="내용을 입력하세요"
                    ></textarea>
                  </td>
                </tr>
              </tbody>
            </table>

            <!-- 질문 버튼 그룹 -->
            <div class="button-group">
              <button
                v-if="showQuestionSave"
                type="button"
                class="btn btn-primary"
                @click="saveQuestion"
              >
                저장
              </button>
              <button
                v-if="showQuestionUpdate"
                type="button"
                class="btn btn-primary"
                @click="updateQuestion"
              >
                수정
              </button>
              <button
                v-if="showQuestionDelete"
                type="button"
                class="btn btn-secondary"
                @click="deleteQuestion"
              >
                삭제
              </button>
              <button
                type="button"
                class="btn btn-cancel"
                @click="modalState.$patch({ isOpen: false })"
              >
                취소
              </button>
            </div>
          </div>

          <!-- 답변 섹션 -->
          <div v-if="modalState.type === 'update'" class="answer-section">
            <h3 class="section-title">댓글</h3>

            <table class="form-table">
              <tbody>
                <tr>
                  <th class="form-label">작성자</th>
                  <td class="form-input">
                    <input v-model="qnaAnswerWriter" type="text" class="input-field" readonly />
                  </td>
                </tr>
                <tr>
                  <th class="form-label">내용</th>
                  <td class="form-input">
                    <textarea
                      v-model="qnaAnswer"
                      rows="4"
                      class="textarea-field"
                      :readonly="isReadonlyAnswer"
                      placeholder="답변을 입력하세요"
                    ></textarea>
                  </td>
                </tr>
                <tr>
                  <th class="form-label">답변일</th>
                  <td class="form-input">
                    <input v-model="qnaAnswerDate" type="text" class="input-field" readonly />
                  </td>
                </tr>
              </tbody>
            </table>

            <!-- 답변 버튼 그룹 -->
            <div class="button-group">
              <button v-if="showAnswerSave" type="button" class="btn btn-info" @click="saveAnswer">
                등록
              </button>
              <button
                v-if="showAnswerUpdate"
                type="button"
                class="btn btn-info"
                @click="updateAnswer"
              >
                수정
              </button>
              <button
                v-if="showAnswerDelete"
                type="button"
                class="btn btn-secondary"
                @click="deleteAnswer"
              >
                삭제
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<style>
@import './styled.css';
</style>
