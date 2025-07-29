<script setup>
import { onMounted, ref } from 'vue';
import { useUserInfo } from '@/stores/loginInfoState';
import axios from 'axios';
import { ElMessage } from 'element-plus';
import { useModalState } from '@/stores/modalState';

const props = defineProps({
  counselId: {
    type: Number,
    default: 0,
  },
  lectures: {
    type: Object,
    default: () => {},
  },
});

const emits = defineEmits(['success']);

const { user } = useUserInfo();
const modalState = useModalState();

const counselDetail = ref({
  lecId: '',
});

const showSaveBtn = ref(true);
const showDeleteBtn = ref(false);
const showUpdateBtn = ref(false);

const counselStudentNameInput = ref(null);
const counselTitleInput = ref(null);
const counselSelectorInput = ref(null);
const counselContentInput = ref(null);

const urlParamForUpdateSave = () => {
  const counsel = counselDetail.value;
  const param = {
    counselId: counsel.counselId,
    counselStudentName: counsel.counselStudentName,
    counselTitle: counsel.counselTitle,
    counselContent: counsel.counselContent,
    counselLecId: counsel.lecId,
  };

  if (!param.counselLecId) {
    ElMessage.error('과목을 선택해주세요.');
    counselSelectorInput.value.focus();
    return;
  }
  if (!param.counselStudentName) {
    ElMessage.error('학생이름을 입력해주세요.');
    counselStudentNameInput.value.focus();
    return;
  }
  if (!param.counselTitle) {
    ElMessage.error('제목을 입력해주세요.');
    counselTitleInput.value.focus();
    return;
  }
  if (!param.counselContent) {
    ElMessage.error('내용을 입력해주세요.');
    counselContentInput.value.focus();
    return;
  }

  return new URLSearchParams(param);
};

const emitAndCloseModal = (emit) => {
  emits(emit);
  modalState.$patch({ isOpen: false });
};

const searchCounselDetail = () => {
  const param = {
    counselId: props.counselId,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/counselDetail.do', urlParam)
    .then((res) => {
      console.log(res, '!!!!!!!!!!!!!!!!!!!!!!!! 추후 삭제 예정!!!!!!!!!!!!!!!!!!!!!!!');
      console.log('상담 세부사항 불러오기 성공');
      if (user.userType.toUpperCase() === 'M') {
        showUpdateBtn.value = true;
        showDeleteBtn.value = true;
        showSaveBtn.value = false;
      }
      counselDetail.value = res.data.detailValue;
      console.log(counselDetail.value);
    })
    .catch((err) => {
      ElMessage.error('서버 오류 발생');
      console.log('상담 세부사항 서버 오류 발생 err: ', err);
    });
};

const saveCounselDetail = () => {
  const urlParam = urlParamForUpdateSave();

  axios
    .post('/api/manage/counselSave.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        ElMessage.success('저장 완료하였습니다');
        emitAndCloseModal('success');
      } else {
        ElMessage.error('저장 실패하였습니다.');
      }
    })
    .catch((err) => {
      console.log('상담 세부사항 저장 서버 오류 발생 err: ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const updateCounselDetail = () => {
  const urlParam = urlParamForUpdateSave();

  axios
    .post('/api/manage/counselUpdate.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        console.log('상담 세부사항 수정 성공!');
        ElMessage.success('수정되었습니다.');
        emitAndCloseModal('success');
      } else {
        console.log('상담 세부사항 수정 실패!');
        ElMessage.error('수정에 실패하였습니다.');
      }
    })
    .catch((err) => {
      console.log('상담 세부사항 수정 서버 오류 err : ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const deleteCounselDetail = () => {
  const param = {
    counselId: props.counselId,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/counselDelete.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        console.log('상담 세부사항 삭제 성공!');
        ElMessage.success('삭제 완료하였습니다.');
        emitAndCloseModal('success');
      } else {
        console.log('상담 세부사항 삭제 실패!');
        ElMessage.error('삭제 실패하였습니다.');
      }
    })
    .catch((err) => {
      console.log('상담 세부사항 삭제 서버 오류 err : ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const closeCounselDetail = () => {
  emitAndCloseModal('success');
};

onMounted(() => {
  modalState.type === 'counselDetail' && searchCounselDetail();
});
</script>
<template>
  <Teleport to="body">
    <div class="counsel-modal-backdrop">
      <div class="counsel-modal-content">
        <div class="counsel-detail-container">
          <div class="counsel-detail-title">상담관리</div>
          <button class="close-btn" @click="closeCounselDetail">x</button>

          <div class="counsel-detail-form">
            <div class="counsel-detail-form-row">
              <div class="counsel-detail-field"></div>
              <div class="counsel-detail-info">
                <select
                  id="counsel-detail-subject-selector"
                  ref="counselSelectorInput"
                  v-model="counselDetail.lecId"
                >
                  <option value="">-- 과목을 선택하세요 --</option>
                  <option
                    v-for="lecture in props.lectures"
                    :key="lecture.lecId"
                    :value="lecture.lecId"
                    class="counsel-detail-subject-option"
                  >
                    {{ lecture.lecName }}
                  </option>
                </select>
              </div>
            </div>
            <div class="counsel-detail-form-row">
              <div class="counsel-detail-field"></div>
              <div ref="counselStudentNameInput" class="counsel-detail-info">
                <input
                  id="counsel-student-name-input"
                  v-model.lazy="counselDetail.counselStudentName"
                  type="text"
                />
              </div>
            </div>
            <div class="counsel-detail-form-row">
              <div class="counsel-detail-field"></div>
              <div ref="counselTitleInput" class="counsel-detail-info">
                <input
                  id="counsel-student-title-input"
                  v-model.lazy="counselDetail.counselTitle"
                  type="text"
                />
              </div>
            </div>
            <div class="counsel-detail-form-row">
              <div class="counsel-detail-field"></div>
              <div ref="counselContentInput" class="counsel-detail-info">
                <input
                  id="counsel-student-content-input"
                  v-model.lazy="counselDetail.counselContent"
                  type="text"
                />
              </div>
            </div>
          </div>
          <div class="counsel-btn-container">
            <button v-if="showSaveBtn" id="counsel-save-btn" @click="saveCounselDetail"></button>
            <button
              v-if="showUpdateBtn"
              id="counsel-update-btn"
              @click="updateCounselDetail"
            ></button>
            <button
              v-if="showDeleteBtn"
              id="counsel-delete-btn"
              @click="deleteCounselDetail"
            ></button>
            <button id="counsel-cancel-btn" @click="closeCounselDetail"></button>
          </div>
        </div>
      </div>
    </div>
  </Teleport>
</template>
<style>
@import './styled.css';
</style>
