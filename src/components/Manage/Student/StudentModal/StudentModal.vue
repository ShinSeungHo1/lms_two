<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref } from 'vue';

const modalState = useModalState();

const props = defineProps({
  studentId: {
    type: String,
    default: '',
  },
});

const studentDetail = ref({});
const lectureList = ref([]);

const headerList = ['강의번호', '강의명', '개강 일자', '종강 일자'];

const searchStudentDetail = () => {
  const url = `/api/manage/student-detail/${props.studentId}`;

  axios
    .post(url)
    .then((res) => {
      console.log('학생 세부사항 불러오기 성공');
      console.log(res);
      studentDetail.value = res.data;
      lectureList.value = res.data.lectureInfo;
      if (studentDetail.value.statusYn === 'Y') {
        studentDetail.value.statusYn = '재학';
      } else if (studentDetail.value.statusYn === 'N') {
        studentDetail.value.statusYn = '탈퇴';
      } else {
        studentDetail.value.statusYn = '대기';
      }
      lectureList.value.forEach((item) => {
        item.lectureStartDate = formatDate(item.lectureStartDate);
        item.lectureEndDate = formatDate(item.lectureEndDate);
      });
    })
    .catch((err) => {
      console.log('학생 세부사항 불러오기 실패 err: ', err);
    });
};

const formatDate = (timestamp) => {
  const date = new Date(Number(timestamp));
  return date.toLocaleDateString();
};

const closeModal = () => {
  modalState.$patch({ isOpen: false });
};

onMounted(() => {
  searchStudentDetail();
});
</script>

<template>
  <Teleport to="body">
    <div class="modal-backdrop">
      <div class="modal-content">
        <div class="student-detail-title">
          학생 상세
          <button class="close-mark" @click="closeModal">x</button>
        </div>
        <div class="student-detail-container">
          <div class="student-detail-form-label">
            <div class="student-field-label">학생ID</div>
            <div class="student-info-label">{{ studentDetail.studentId }}</div>
            <div class="student-field-label">이름</div>
            <div class="student-info-label">{{ studentDetail.studentName }}</div>
            <div class="student-field-label">학번</div>
            <div class="student-info-label">{{ studentDetail.studentId }}</div>
            <div class="student-field-label">연락처</div>
            <div class="student-info-label">{{ studentDetail.studentId }}</div>
            <div class="student-field-label">이메일</div>
            <div class="student-info-label">{{ studentDetail.studentEmail }}</div>
            <div class="student-field-label">생일</div>
            <div class="student-info-label">
              {{ studentDetail.studentBirthday?.substring(0, 10) }}
            </div>
            <div class="student-field-label">재학상태</div>
            <div class="student-info-label">{{ studentDetail.statusYn }}</div>
            <div class="student-field-label">취업 상태</div>
            <div class="student-info-label">
              {{ studentDetail.studentEmpStatus === 'Y' ? '취업' : '미취업' }}
            </div>
          </div>
        </div>
        강의 목록
        <div class="student-detail-table-container">
          <table class="student-detail-table">
            <thead class="student-detail-table-header">
              <tr class="student-detail-table-header-row">
                <th
                  v-for="(header, index) in headerList"
                  :key="`h-${index}`"
                  class="student-detail-table-header-cell"
                >
                  {{ header }}
                </th>
              </tr>
            </thead>
            <tbody class="student-detail-table-body">
              <tr
                v-for="lecture in lectureList"
                :key="lecture.lecId"
                class="student-detail-table-body-row"
              >
                <td class="student-detail-table-body-cell">{{ lecture.lecId }}</td>
                <td class="student-detail-table-body-cell">{{ lecture.lectureName }}</td>
                <td class="student-detail-table-body-cell">{{ lecture.lectureStartDate }}</td>
                <td class="student-detail-table-body-cell">{{ lecture.lectureEndDate }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <button class="close-student-detail-button" @click="closeModal"></button>
      </div>
    </div>
  </Teleport>
</template>

<style>
@import './styled.css';
</style>
