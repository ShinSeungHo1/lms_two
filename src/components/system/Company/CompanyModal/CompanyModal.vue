<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { ElMessage } from 'element-plus';
import { onMounted, ref, watch } from 'vue';

const props = defineProps({
  companyId: {
    type: Number,
    default: 0,
  },
});

const emits = defineEmits(['success']);

const showSaveBtn = ref(true);
const showUpdateBtn = ref(false);
const showDeleteBtn = ref(false);

const companyNameInput = ref('');
const companyCeoInput = ref('');
const companyHpInput = ref('');
const zipcodeInput = ref('');
const companyLocInput = ref('');
const detailAddressInput = ref('');
const companyEmailInput = ref('');
const companyRegDateInput = ref('');

const inputRef = ref({
  companyName: companyNameInput,
  companyCeo: companyCeoInput,
  companyHp: companyHpInput,
  zipcode: zipcodeInput,
  companyLoc: companyLocInput,
  detailAddress: detailAddressInput,
  companyEmail: companyEmailInput,
  companyRegDate: companyRegDateInput,
});

const modalState = useModalState();
const companyDetailList = ref({
  companyName: '',
  companyCeo: '',
  companyHp: '',
  zipcode: '',
  companyLoc: '',
  detailAddress: '',
  companyEmail: '',
  companyRegDate: '',
});

const closeModal = () => {
  modalState.$patch({ isOpen: false });
};

const searchCompanyDeatil = () => {
  const param = {
    companyId: props.companyId,
  };

  const urlParam = new URLSearchParams(param);
  axios
    .post('/api/system/companyDetail.do', urlParam)
    .then((res) => {
      console.log('회사 세부사항 정보 불러오기 성공!');

      companyDetailList.value = res.data.detailValue;
      companyDetailList.value.companyRegDate = formatedDate(res.data.detailValue.companyRegDate);
      showDeleteBtn.value = true;
      showUpdateBtn.value = true;
      showSaveBtn.value = false;
    })
    .catch((err) => {
      console.log('회사 세부사항 불러오기 실패 err: ', err);
    });
};

const saveCompany = () => {
  const urlParam = vaildateAndMakeParam();

  axios
    .post('/api/system/companySave.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        ElMessage.success('회사정보 저장 성공!');
        emitAndModalClose('success');
      } else {
        ElMessage.error('회사정보 저장 실패');
      }
    })
    .catch((err) => {
      ElMessage.error('회사정보 저장 실패');
      console.log('저장 error Message : ', err);
    });
};

const updateCompany = () => {
  const urlParam = vaildateAndMakeParam();
  urlParam.append('companyId', props.companyId);

  axios
    .post('/api/system/companyUpdate.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        ElMessage.success('수정 성공하였습니다.');
        emitAndModalClose('success');
      } else {
        ElMessage.error('수정 실패하였습니다.');
      }
    })
    .catch((err) => {
      ElMessage.error('수정 실패하였습니다.');
      console.log('수정 error Message : ', err);
    });
};

const deleteCompany = () => {
  const param = {
    companyId: props.companyId,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/system/companyDelete.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        ElMessage.success('삭제 완료하였습니다.');
        emitAndModalClose('success');
      } else {
        ElMessage.error('삭제 실패하였습니다.');
      }
    })
    .catch((err) => {
      ElMessage.error('삭제 실패 하였습니다.', err);
    });
};

const validation = () => {
  const validations = [
    { field: 'companyName', message: '회사명을 입력해주세요' },
    { field: 'companyCeo', message: '대표명을 입력해주세요' },
    { field: 'companyHp', message: '휴대전화 번호를 입력해주세요' },
    { field: 'zipcode', message: '우편번호를 입력해주세요' },
    { field: 'companyLoc', message: '기본주소를 입력해주세요' },
    { field: 'detailAddress', message: '상세주소를 입력해주세요' },
    { field: 'companyEmail', message: '이메일을 입력해주세요' },
    { field: 'companyRegDate', message: '가입일자를 입력해주세요' },
  ];

  for (let i = 0; i < validations.length; i++) {
    const { field, message } = validations[i];

    if (!companyDetailList.value[field] || companyDetailList.value[field] === '') {
      ElMessage.error(message);
      inputRef.value[field].focus();
      return false;
    }
  }
  return true;
};

const validateEmail = () => {
  const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(companyDetailList.value.companyEmail);

  if (!isValid) {
    ElMessage.error('이메일 형식을 확인하여주세요');
    return false;
  }
  return true;
};

const formatPhoneNumber = () => {
  let numbers = companyDetailList.value.companyHp.replace(/\D/g, '');
  let formatted = '';

  if (numbers.startsWith('02')) {
    // 서울 번호일 경우 (2자리 지역번호)
    if (numbers.length <= 2) {
      formatted = numbers;
    } else if (numbers.length <= 5) {
      formatted = numbers.slice(0, 2) + '-' + numbers.slice(2);
    } else if (numbers.length <= 9) {
      formatted = numbers.slice(0, 2) + '-' + numbers.slice(2, 5) + '-' + numbers.slice(5);
    } else {
      formatted = numbers.slice(0, 2) + '-' + numbers.slice(2, 6) + '-' + numbers.slice(6, 10);
    }
  } else {
    // 휴대폰 번호 또는 일반 번호 (3자리 지역번호)
    if (numbers.length <= 3) {
      formatted = numbers;
    } else if (numbers.length <= 7) {
      formatted = numbers.slice(0, 3) + '-' + numbers.slice(3);
    } else if (numbers.length <= 11) {
      formatted = numbers.slice(0, 3) + '-' + numbers.slice(3, 7) + '-' + numbers.slice(7, 11);
    } else {
      formatted = numbers.slice(0, 3) + '-' + numbers.slice(3, 7) + '-' + numbers.slice(7, 11);
    }
  }

  companyDetailList.value.companyHp = formatted;
};

const findAddressDaum = () => {
  new window.daum.Postcode({
    onComplete: function (data) {
      companyDetailList.value.zipcode = data.zonecode;
      companyDetailList.value.companyLoc = data.address;
    },
  }).open();
};

// const displayDate = computed({
//   get() {
//     if (!companyDetailList.value.companyRegDate) {
//       return '';
//     }
//     console.log(new Date(companyDetailList.value.companyRegDate).toISOString());
//     return new Date(companyDetailList.value.companyRegDate).toISOString().split('T')[0];
//   },
//   set(value) {
//     if (value) {
//       return (companyDetailList.value.companyRegDate = new Date(value).getTime());
//     }
//   },
// });

const emitAndModalClose = (emit) => {
  emits(emit);
  modalState.$patch({ isOpen: false });
};

const vaildateAndMakeParam = () => {
  if (!validation()) {
    return;
  }

  if (!validateEmail()) {
    return;
  }

  const param = {
    companyName: companyDetailList.value.companyName,
    companyCeo: companyDetailList.value.companyCeo,
    companyHp: companyDetailList.value.companyHp,
    companyLoc: companyDetailList.value.zipcode,
    detailName: companyDetailList.value.companyLoc,
    companyDetailAddress: companyDetailList.value.detailAddress,
    companyEmail: companyDetailList.value.companyEmail,
    companyRegDate: companyDetailList.value.companyRegDate,
  };

  return new URLSearchParams(param);
};

const formatedDate = (input) => {
  if (input === null) {
    return;
  } else {
    const date = new Date(input);
    const year = date.getFullYear();
    const month = ('0' + (date.getMonth() + 1)).slice(-2);
    const day = ('0' + date.getDate()).slice(-2);
    return year + '-' + month + '-' + day;
  }
};

watch(
  () => companyDetailList.value.companyHp,
  () => {
    formatPhoneNumber();
  },
);

onMounted(() => {
  modalState.type === 'detailCompany' && searchCompanyDeatil();
});
</script>
<template>
  <div class="modal-company-overlay" @click="closeModal">
    <div class="modal-company-container" @click.stop>
      <!-- 헤더 -->
      <div class="modal-company-header">
        <h2>회사 정보</h2>
        <button class="close-company-btn" @click="closeModal">×</button>
      </div>

      <!-- 컨텐츠 -->
      <div class="modal-company-content">
        <div class="company-content-row">
          <div class="company-menu-cell">회사명</div>
          <div class="company-input-cell">
            <input
              ref="companyNameInput"
              v-model.lazy="companyDetailList.companyName"
              type="text"
              class="company-name-input"
            />
          </div>
        </div>
        <div class="company-content-row">
          <div class="company-menu-cell">대표명</div>
          <div class="company-input-cell">
            <input
              ref="companyCeoInput"
              v-model.lazy="companyDetailList.companyCeo"
              type="text"
              class="company-ceo-name-input"
            />
          </div>
        </div>
        <div class="company-content-row">
          <div class="company-menu-cell">휴대전화</div>
          <div class="company-input-cell">
            <input
              ref="companyHpInput"
              v-model="companyDetailList.companyHp"
              type="text"
              class="company-hp-input"
              placeholder="전화번호 입력"
            />
          </div>
        </div>
        <div class="company-content-row">
          <div class="company-menu-cell">우편번호</div>
          <div class="company-input-cell">
            <input
              ref="zipcodeInput"
              v-model="companyDetailList.zipcode"
              type="text"
              class="company-post-input"
              placeholder="우편번호"
              readonly
            />
            <button class="company-address-search-btn" @click="findAddressDaum">주소 검색</button>
          </div>
        </div>
        <div class="company-content-row">
          <div class="company-menu-cell">기본주소</div>
          <div class="company-input-cell">
            <input
              ref="companyLocInput"
              v-model.lazy="companyDetailList.companyLoc"
              type="text"
              class="company-basic-post-input"
              placeholder="기본주소"
              readonly
            />
          </div>
        </div>
        <div class="company-content-row">
          <div class="company-menu-cell">상세주소</div>
          <div class="company-input-cell">
            <input
              ref="detailAddressInput"
              v-model.lazy="companyDetailList.detailAddress"
              type="text"
              class="company-detaili-post-input"
              placeholder="상세주소"
            />
          </div>
        </div>
        <div class="company-content-row">
          <div class="company-menu-cell">이메일</div>
          <div class="company-input-cell">
            <input
              ref="companyEmailInput"
              v-model.lazy="companyDetailList.companyEmail"
              type="text"
              class="company-email-input"
              placeholder="example@example.com"
            />
          </div>
        </div>
        <div class="company-content-row">
          <div class="company-menu-cell">가입일자</div>
          <div class="company-input-cell">
            <input
              ref="companyRegDateInput"
              v-model="companyDetailList.companyRegDate"
              type="date"
              class="company-join-date-input"
            />
          </div>
        </div>
      </div>

      <!-- 버튼 -->
      <div class="modal-company-footer">
        <button v-if="showSaveBtn" class="confirm-company-btn" @click="saveCompany">저장</button>
        <button v-if="showUpdateBtn" class="update-company-btn" @click="updateCompany">수정</button>
        <button v-if="showDeleteBtn" class="delete-company-btn" @click="deleteCompany">삭제</button>
        <button class="close-company-btn" @click="closeModal">닫기</button>
      </div>
    </div>
  </div>
</template>
<style>
@import './styled.css';
</style>
