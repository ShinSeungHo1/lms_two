<script setup>
import { useModalStateSub } from '@/stores/modalState';
import axios from 'axios';
import { ElMessage } from 'element-plus';

import { onMounted, ref } from 'vue';

const props = defineProps({
  empId: {
    type: Number,
    default: 0,
  },
  loginId: {
    type: String,
    default: '',
  },
});

const modalStateSub = useModalStateSub();

const studentDetail = ref({});

const showSaveBtn = ref(true);
const showUpdateBtn = ref(false);

const recruitEmpNameInput = ref(null);
const recruitJoinDateInput = ref(null);

const emits = defineEmits(['success']);

const emitsAndModalClose = (emit) => {
  emits(emit);
  modalStateSub.$patch({ isOpen: false });
};

const searchStudentRecruitDetail = () => {
  const param = {
    empId: props.empId,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/empStudentDetail.do', urlParam)
    .then((res) => {
      console.log('학생 취업 상세정보 불러오기 성공');
      console.log(res);
      studentDetail.value = res.data.detailValue;
      studentDetail.value.empJoinDate = formatDate(studentDetail.value.empJoinDate);
      studentDetail.value.empRetireDate = formatDate(studentDetail.value.empRetireDate);
      showSaveBtn.value = false;
      showUpdateBtn.value = true;
    })
    .catch((err) => {
      console.log('학생 취업 상세정보 서버 오류 발생 err : ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const saveStudentRecruit = () => {
  if (!validateRecruit()) return;

  const detail = studentDetail.value;
  const param = {
    loginID: props.loginId,
    empName: detail.empName,
    empJoinDate: detail.empJoinDate,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/RecruitSave.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        ElMessage.success('저장되었습니다.');
        emitsAndModalClose('success');
      } else if (res.data.result === 'retirefail') {
        ElMessage.error('입사와 퇴직일자를 확인해주세요');
        return;
      } else {
        ElMessage.error('재직여부를 확인해주세요');
        return;
      }
    })
    .catch((err) => {
      console.log('힉생 취업 상세정보 저장 서버 오류 발생 err : ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const updateStudentRecruit = () => {
  const detail = studentDetail.value;
  if (!validateRecruit()) return;

  if (detail.empJoinDate > detail.empRetireDate) {
    ElMessage.error('퇴직일자가 입사일보다 빠를 수 없습니다.');
    return;
  }

  const param = {
    empId: props.empId,
    empName: detail.empName,
    empJoinDate: reFormDate(detail.empJoinDate),
    empRetireDate: detail.empRetireDate ? reFormDate(detail.empRetireDate) : '',
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/empStudentUpdate.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        console.log('학생 취업 상세정보 수정 성공!');
        ElMessage.success('수정 되었습니다.');
        emitsAndModalClose('success');
      } else {
        ElMessage.error('입사와 퇴직일자를 확인해주세요');
        return;
      }
    })
    .catch((err) => {
      console.log('학생 취업 상세정보 수정 서버 오류 발생 err : ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const reFormDate = (date) => {
  return new Date(date).toISOString().slice(0, 19).replace('T', ' ');
};

const formatDate = (timestamp) => {
  if (timestamp) {
    const date = new Date(timestamp);
    const year = date.getFullYear();
    const month = ('0' + (date.getMonth() + 1)).slice(-2);
    const day = ('0' + date.getDate()).slice(-2);
    return `${year}-${month}-${day}`;
  }
};

const validateRecruit = () => {
  if (!studentDetail.value.empName) {
    ElMessage.error('업체명을 입력해주세요');
    recruitEmpNameInput.value.focus();
    return false;
  }

  if (!studentDetail.value.empJoinDate) {
    ElMessage.error('입사일을 입력해주세요');
    recruitJoinDateInput.value.focus();
    return false;
  }

  return true;
};

const closeModal = () => {
  modalStateSub.$patch({ isOpen: false });
};

onMounted(() => {
  modalStateSub.type === 'updateRecruit' && searchStudentRecruitDetail();
});
</script>
<template>
  <Teleport to="body">
    <div class="recruit-modal-backdrop">
      <div class="recruit-modal-content">
        <div class="recruit-modal-header">
          <div class="recruit-modal-title">학생 정보 상세보기</div>
          <button id="recruit-x-btn" @click="closeModal">x</button>
        </div>

        <div class="recruit-modal-form">
          <div class="recruit-modal-form-row">
            <div class="recruit-modal-field">기업명</div>
            <div class="recruit-modal-info">
              <input
                id="recruit-modal-company-name"
                ref="recruitEmpNameInput"
                v-model.lazy="studentDetail.empName"
                type="text"
              />
            </div>
          </div>
          <div class="recruit-modal-form-row">
            <div class="recruit-modal-field">입사일</div>
            <div class="recruit-modal-info">
              <input
                ref="recruitJoinDateInput"
                v-model="studentDetail.empJoinDate"
                class="recruit-modal-date"
                type="date"
              />
            </div>
          </div>
          <div v-if="modalStateSub.type === 'updateRecruit'" class="recruit-modal-form-row">
            <div class="recruit-modal-field">퇴사일</div>
            <div class="recruit-modal-info">
              <input v-model="studentDetail.empRetireDate" class="recruit-modal-date" type="date" />
            </div>
          </div>
        </div>
        <div class="recruit-modal-btn-container">
          <button v-if="showSaveBtn" id="recruit-save-btn" @click="saveStudentRecruit">저장</button>
          <button v-if="showUpdateBtn" id="recruit-update-btn" @click="updateStudentRecruit">
            수정
          </button>
          <button id="recruit-cancel-btn" @click="closeModal">취소</button>
        </div>
      </div>
    </div>
  </Teleport>
</template>
<style>
@import './styled.css';
</style>
