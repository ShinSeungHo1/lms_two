<script setup>
import axios from 'axios';
import { ElMessage, ElMessageBox } from 'element-plus';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { useModalState } from '@/stores/modalState';

import PageNavigation from '@/components/common/PageNavigation.vue';
import StudentModal from '../StudentModal/StudentModal.vue';

const headerList = ['학번', '이름', '연락처', '가입일자', '취업여부', '재학상태'];

const route = useRoute();
const modalState = useModalState();

const studentList = ref([]);
const listCount = ref(0);

const studentId = ref('');

const searchStudentList = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/studentListBody.do', urlParam)
    .then((res) => {
      console.log(res);
      console.log('학생 리스트 불러오기 성공!');
      studentList.value = res.data.list;
      listCount.value = res.data.count;
    })
    .catch((err) => {
      console.log('학생 리스트 불러오기 실패 err: ', err);
    });
};

const changeStatus = (id, prevStatus, changedStatus) => {
  ElMessageBox.confirm('재학 상태를 변경하시겠습니까?', {
    confirmButtonText: '변경',
    cancelButtonText: '취소',
    type: 'warning',
  })
    .then(() => {
      const param = {
        studentId: id,
        studentStatus: changedStatus,
      };

      axios
        .post('/api/manage/student-status', param)
        .then((res) => {
          if (res.data === 'SUCCESS') {
            ElMessage.success('재학 상태가 변경되었습니다.');
            searchStudentList();
          } else {
            ElMessage.error('변경에 실패하였습니다.');
          }
        })
        .catch((err) => {
          ElMessage.error('서버 오류 발생 err : ', err);
        });
    })
    .catch(() => {
      ElMessage.info('재학 상태 변경 취소');
      searchStudentList();
    });
};

const studentDetailModal = (id) => {
  modalState.$patch({ isOpen: true, type: 'studentDetail' });
  studentId.value = id;
};

watch(
  () => route.query,
  () => {
    searchStudentList();
  },
);

onMounted(() => {
  searchStudentList();
});
</script>

<template>
  <div class="student-table-container">
    <table class="student-table">
      <thead class="student-table-header">
        <tr class="student-table-header-row">
          <th
            v-for="(header, index) in headerList"
            :key="`h-${index}`"
            class="student-table-header-cell"
          >
            {{ header }}
          </th>
        </tr>
      </thead>
      <tbody class="student-table-body">
        <tr v-for="student in studentList" :key="student.studentId" class="student-table-body-row">
          <td class="student-table-body-cell">
            {{ student.studentNumber }}
          </td>
          <td class="student-table-body-cell" @click="studentDetailModal(student.studentId)">
            {{ student.studentName }}
          </td>
          <td class="student-table-body-cell">
            {{ student.studentHp }}
          </td>
          <td class="student-table-body-cell">
            {{ student.studentRegDate?.substring(0, 10) }}
          </td>
          <td class="student-table-body-cell">
            {{ student.studentEmpStatus === 'Y' ? '취업' : '미취업' }}
          </td>
          <td class="student-table-body-cell">
            <select
              id="student-change-status"
              :value="student.statusYN"
              @change="changeStatus(student.studentId, student.statusYN, $event.target.value)"
            >
              <option value="Y">재학</option>
              <option value="N">탈퇴</option>
              <option value="W">승인 대기중</option>
            </select>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation
    :total-items="listCount"
    :items-per-page="5"
    :on-page-change="searchStudentList"
  />
  <StudentModal
    v-if="modalState.type === 'studentDetail' && modalState.isOpen"
    :student-id="studentId"
  />
</template>

<style>
@import './styled.css';
</style>
