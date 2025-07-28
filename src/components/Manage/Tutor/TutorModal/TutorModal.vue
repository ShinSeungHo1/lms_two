<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';

const headList = ['강의 번호', '강의명', '개강 일자', '종강 일자'];

const modalState = useModalState();

const tutorDetail = ref({});
const tutorLectureInfo = ref([]);

const props = defineProps({
  tutorId: {
    type: String,
    default: '',
  },
});

const searchTutorDetail = () => {
  const url = `/api/manage/tutor-detail/${props.tutorId}`;
  axios
    .post(url)
    .then((res) => {
      console.log('강사 세부사항 불러오기 성공');
      tutorDetail.value = res.data;
      tutorLectureInfo.value = res.data.lectureInfo;

      if (tutorDetail.value.insStatusYn === 'Y') {
        tutorDetail.value.insStatusYn = '재직';
      } else if (tutorDetail.value.insStatusYn === 'W') {
        tutorDetail.value.insStatusYn = '대기';
      } else {
        tutorDetail.value.insStatusYn = '퇴직';
      }
    })
    .catch((err) => {
      ElMessage.error('강사 세부사항 서버 오류 발생');
      console.log('서버 오류 발생 err : ', err);
    });
};

const closeDetailModal = () => {
  modalState.$patch({ isOpen: false });
};

onMounted(() => {
  searchTutorDetail();
});
</script>

<template>
  <Teleport to="body">
    <div class="modal-backdrop">
      <div class="modal-content">
        <div class="tutor-detail-title">
          강사 상세
          <button class="close-btn" @click="closeDetailModal">x</button>
        </div>
        <div class="tutor-detail-container">
          <div class="tutor-detail-form-label">
            <div class="tutor-field-label"></div>
            <div class="tutor-info-label">{{ tutorDetail.insId }}</div>
            <div class="tutor-field-label"></div>
            <div class="tutor-info-label">{{ tutorDetail.insName }}</div>
            <div class="tutor-field-label"></div>
            <div class="tutor-info-label">{{ tutorDetail.insNumber }}</div>
            <div class="tutor-field-label"></div>
            <div class="tutor-info-label">{{ tutorDetail.insHp }}</div>
            <div class="tutor-field-label"></div>
            <div class="tutor-info-label">{{ tutorDetail.insEmail }}</div>
            <div class="tutor-field-label"></div>
            <div class="tutor-info-label">{{ tutorDetail.insStatusYn }}</div>
            <div class="tutor-field-label"></div>
            <div class="tutor-info-label">{{ tutorDetail.insBank }}</div>
            <div class="tutor-field-label"></div>
            <div class="tutor-info-label">{{ tutorDetail.insAccount }}</div>
          </div>
        </div>
        강의 목록
        <div class="tutor-lecture-table-container">
          <table class="tutor-lecture-table">
            <thead class="tutor-lecture-table-head">
              <tr class="tutor-lecture-table-head-row">
                <th
                  v-for="(head, index) in headList"
                  :key="index"
                  class="tutor-lecture-table-head-cell"
                >
                  {{ head }}
                </th>
              </tr>
            </thead>
            <tbody v-if="tutorLectureInfo.length > 0" class="tutor-lecture-table-body">
              <tr
                v-for="lecture in tutorLectureInfo"
                :key="lecture.lecId"
                class="tutor-lecture-table-body-row"
              >
                <td class="tutor-lecture-table-body-cell">
                  {{ lecture.lecId }}
                </td>
                <td class="tutor-lecture-table-body-cell">
                  {{ lecture.lecName }}
                </td>
                <td class="tutor-lecture-table-body-cell">
                  {{ lecture.lecStartDate?.substring(0, 10) }}
                </td>
                <td class="tutor-lecture-table-body-cell">
                  {{ lecture.lecEndDate?.substring(0, 10) }}
                </td>
              </tr>
            </tbody>
            <tbody v-else class="tutor-lecture-empty">
              <tr class="tutor-lecture-empty-row">
                <td class="tutor-lecture-empty-cell" colspan="4">진행하는 강의 목록이 없습니다.</td>
              </tr>
            </tbody>
          </table>
        </div>
        <button class="close-tutor-detail-btn" @click="closeDetailModal"></button>
      </div>
    </div>
  </Teleport>
</template>

<style>
@import './styled.css';
</style>
