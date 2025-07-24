<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref } from 'vue';
import { useUserInfo } from '@/stores/loginInfoState';
import { ElMessage } from 'element-plus';

const { user } = useUserInfo();

const modalState = useModalState();

const props = defineProps({
  mtrId: {
    type: Number,
    default: 0,
  },
  lecId: {
    type: Number,
    default: 0,
  },
  lecName: {
    type: String,
    default: '',
  },
  lectureList: {
    type: Array,
    default: () => [],
  },
});

const validation = (lecName, mtrTitle, mtrContent, mtrFileInput) => {
  if (!lecName) {
    ElMessage.error('강의명을 선택해주세요');
    return false;
  }

  if (!mtrTitle) {
    ElMessage.error('제목을 입력해주세요');
    return false;
  }

  if (!mtrContent) {
    ElMessage.error('내용을 입력해주세요');
    return false;
  }

  if (!mtrFileInput[0]) {
    ElMessage.error('파일을 업로드해주세요');
    return false;
  }

  return true;
};

const createFileData = (mtrFileInput, lecId, lecName, mtrTitle, mtrContent, materiId) => {
  const fileData = new FormData();
  if (mtrFileInput.length > 0) {
    fileData.append('materiMaterial', mtrFileInput[0].name);
    fileData.append('file', mtrFileInput[0]);
  }

  fileData.append('lecId', lecId);
  fileData.append('lecName', lecName);
  fileData.append('mtrTitle', mtrTitle);
  fileData.append('mtrContent', mtrContent);
  fileData.append('materiId', materiId);

  return fileData;
};

const detailValue = ref({});
const mtrUrl = ref('');
const lecList = props.lectureList;
const selectFile = ref([]);

const showSaveMtrBtn = ref(false);
const showUpdateMtrBtn = ref(false);
const showDeleteMtrBtn = ref(false);
const showMtrFileInput = ref(true);

const isTitleReadonly = ref(false);
const isContentReadonly = ref(false);
const isDropdownOpen = ref(false);
const canClickLecName = ref(true);

const emits = defineEmits(['success']);

const modalEmitFunc = (emit) => {
  modalState.$patch({ isOpen: false });
  emits(emit);
};

const toggleDropdown = () => {
  console.log('toggleDropDown 실행');
  isDropdownOpen.value = !isDropdownOpen.value;
};

const searchMtrDetail = () => {
  console.log(props);
  const param = {
    materiId: props.mtrId,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/support/getMtrDetail.do', urlParam, { responseEncoding: 'utf-8' })
    .then((res) => {
      console.log('Detail 불러오기 성공!');
      console.log(res);
      detailValue.value = res.data.detailValue;
      mtrUrl.value = res.data.materiMaterialUrl;

      if (user.userType === 'S') {
        canClickLecName.value = false;
        isTitleReadonly.value = true;
        isContentReadonly.value = true;
        showMtrFileInput.value = false;
      }
      if (user.userType === 'T') {
        console.log(canClickLecName.value);
        showUpdateMtrBtn.value = true;
        showDeleteMtrBtn.value = true;
      }
      if (user.userType === 'M') {
        canClickLecName.value = false;
        showDeleteMtrBtn.value = true;
        isTitleReadonly.value = true;
        isContentReadonly.value = true;
        showMtrFileInput.value = false;
      }
    })
    .catch(() => {
      console.log('Detail 불러오기 실패!');
    });
};

const selectLecture = (lecture) => {
  detailValue.value.lecId = lecture.lecId;
  detailValue.value.lecName = lecture.lecName;
  isDropdownOpen.value = false;
};

const handleFileSelect = (event) => {
  selectFile.value = event.target.files;
};

const saveMtr = () => {
  const lecId = detailValue.value.lecId;
  const lecName = detailValue.value.lecName;
  const mtrTitle = detailValue.value.materiTitle;
  const mtrContent = detailValue.value.materiContent;
  const mtrFileInput = selectFile.value;
  const materiId = detailValue.value.materiId;

  if (!validation(lecName, mtrTitle, mtrContent, mtrFileInput)) return;

  const createdFileData = createFileData(
    mtrFileInput,
    lecId,
    lecName,
    mtrTitle,
    mtrContent,
    materiId,
  );

  console.log(mtrFileInput);

  axios.post('/api/support/saveMtr.do', createdFileData).then((res) => {
    if (res.data.result === 'success') {
      ElMessage.success('저장 성공!');
      modalEmitFunc('success');
    }
  });
};

const updateMtr = () => {
  const lecId = detailValue.value.lecId;
  const lecName = detailValue.value.lecName;
  const mtrTitle = detailValue.value.materiTitle;
  const mtrContent = detailValue.value.materiContent;
  const mtrFileInput = selectFile.value;
  const materiId = detailValue.value.materiId;

  if (!validation(lecName, mtrTitle, mtrContent, mtrFileInput)) return;

  const createdFileData = createFileData(
    mtrFileInput,
    lecId,
    lecName,
    mtrTitle,
    mtrContent,
    materiId,
  );

  axios
    .post('/api/support/updateMtr.do', createdFileData, {
      headers: {
        'Content-Type': 'multipart/form-data; charset=UTF-8',
      },
    })
    .then((res) => {
      if (res.data.result === 'success') {
        ElMessage.success('수정 성공');
        modalEmitFunc('success');
      }
    })
    .catch(() => {
      ElMessage.error('수정 실패!');
    });
};

const downloadFile = () => {
  const param = {
    materiId: detailValue.value.materiId,
  };

  const urlParam = new URLSearchParams(param);
  axios.post('/api/support/materiDownload.do', urlParam, { responseType: 'blob' }).then((res) => {
    console.log(res);
    const url = window.URL.createObjectURL(res.data);
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', detailValue.value.fileName);
    document.body.appendChild(link);
    link.click();
    link.remove();
  });
};

const deleteMtr = () => {
  const param = {
    materiId: detailValue.value.materiId,
  };
  const urlParam = new URLSearchParams(param);
  axios
    .post('/api/support/deleteMtr.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        modalEmitFunc('success');
        ElMessage.success('삭제 성공');
      }
    })
    .catch(() => {
      ElMessage.error('삭제 실패');
    });
};

onMounted(() => {
  if (modalState.type === 'detailMtr') {
    searchMtrDetail();
  } else {
    showSaveMtrBtn.value = true;
  }
});
</script>

<template>
  <Teleport to="body">
    <div class="form-overlay" @click.self="closeForm">
      <div class="form-container">
        <!-- Header -->
        <div class="form-header">
          <h2 class="form-title">학습 자료 (파일)</h2>
          <button class="close-btn" @click="closeForm">
            <svg
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              @click="modalState.$patch({ isOpen: false })"
            >
              <path
                d="M18 6L6 18M6 6L18 18"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
              />
            </svg>
          </button>
        </div>

        <!-- Form Content -->
        <div class="form-content">
          <div class="form-group">
            <label class="form-label required">강의명</label>
            <input
              v-model.lazy="detailValue.lecName"
              type="text"
              class="form-input cursor-pointer"
              placeholder="강의명을 입력해주세요"
              readonly
              @click="canClickLecName && toggleDropdown()"
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
              <div v-if="lecList.length === 0" class="lecture-dropdown-empty">강의가 없습니다.</div>
            </div>
          </div>

          <div class="form-group">
            <label class="form-label required">제목</label>
            <input
              v-model.lazy="detailValue.materiTitle"
              type="text"
              class="form-input"
              placeholder="제목을 입력해주세요"
              :readonly="isTitleReadonly"
            />
          </div>

          <div class="form-group">
            <label class="form-label required">내용</label>
            <textarea
              v-model.lazy="detailValue.materiContent"
              class="form-textarea"
              placeholder="내용을 입력해주세요"
              rows="4"
              :readonly="isContentReadonly"
            ></textarea>
          </div>

          <div class="form-group">
            <label class="form-label required">파일</label>
            <div class="file-input-wrapper">
              <input
                v-if="showMtrFileInput"
                type="file"
                class="file-input-simple"
                multiple
                @change="handleFileSelect"
              />
              <div class="current-file-info cursor-pointer">
                <span class="current-file-label">현재 파일:</span>
                <span class="current-file-name" @click="downloadFile">{{
                  detailValue.fileName ? detailValue.fileName : '파일이 없습니다.'
                }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Form Actions -->
        <div class="form-actions">
          <button class="btn btn-secondary" @click="modalState.$patch({ isOpen: false })">
            취소
          </button>
          <button v-if="showDeleteMtrBtn" class="btn btn-secondary" @click="deleteMtr">삭제</button>
          <button v-if="showUpdateMtrBtn" class="btn btn-primary" @click="updateMtr">수정</button>
          <button v-if="showSaveMtrBtn" class="btn btn-secondary" @click="saveMtr">저장</button>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<style>
@import './styled.css';
</style>
