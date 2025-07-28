<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { ElMessage, ElMessageBox } from 'element-plus';

import PageNavigation from '@/components/common/PageNavigation.vue';
import TutorModal from '../TutorModal/TutorModal.vue';

const headList = ['강사번호', '이름', '연락처', '가입일자', '재직상태'];

const modalState = useModalState();
const route = useRoute();

const tutorList = ref([]);
const listCount = ref(0);

const tutorId = ref('');

const searchTutorList = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);

  axios.post('/api/manage/tutorListBody.do', urlParam).then((res) => {
    console.log('강사 리스트 불러오기 성공');
    tutorList.value = res.data.list;
    listCount.value = res.data.count;
  });
};

const changeStatus = (id, newStatus) => {
  ElMessageBox.confirm('재직 상태를 변경하시겠습니까?', {
    confirmButtonText: '변경',
    cancelButtonText: '취소',
    type: 'warning',
  })
    .then(() => {
      const param = {
        tutorId: id,
        tutorStatus: newStatus,
      };
      axios
        .post('/api/manage/update-ins-status', param)
        .then((res) => {
          if (res.data === 'SUCCESS') {
            ElMessage.success('재직 상태가 변경되었습니다.');
            searchTutorList();
          } else {
            ElMessage.error('변경 실패하였습니다.');
          }
        })
        .catch((err) => {
          ElMessage.error('서버 오류 발생');
          console.log('서버 오류 발생 err : ', err);
        });
    })
    .catch(() => {
      ElMessage.info('변경 취소하였습니다.');
      searchTutorList();
    });
};

const tutorDetailModal = (id) => {
  modalState.$patch({ isOpen: true, type: 'tutorDetail' });
  tutorId.value = id;
};

watch(
  () => route.query,
  () => {
    searchTutorList();
  },
);

onMounted(() => {
  searchTutorList();
});
</script>
<template>
  <div class="tutro-table-container">
    <table class="tutor-table">
      <thead class="tutor-table-head">
        <tr class="tutor-table-head-row">
          <th v-for="(head, index) in headList" :key="`h-${index}`" class="tutor-table-head-cell">
            {{ head }}
          </th>
        </tr>
      </thead>
      <tbody class="tutor-table-body">
        <tr v-for="tutor in tutorList" :key="tutor.insId" class="tutor-table-body-row">
          <td class="tutor-table-body-cell">
            {{ tutor.insNumber }}
          </td>
          <td class="tutor-table-body-cell" @click="tutorDetailModal(tutor.insId)">
            {{ tutor.insName }}
          </td>
          <td class="tutor-table-body-cell">
            {{ tutor.insHp }}
          </td>
          <td class="tutor-table-body-cell">
            {{ tutor.insRegDate?.substring(0, 10) }}
          </td>
          <td class="tutor-table-body-cell">
            <select
              id="tutor-status-selector"
              :value="tutor.insStatusYn"
              @change="changeStatus(tutor.insId, $event.target.value)"
            >
              <option class="tutor-status-option" value="Y">재직</option>
              <option class="tutor-status-option" value="N">탈퇴</option>
              <option class="tutor-status-option" value="W">승인대기</option>
            </select>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation :items-per-page="5" :total-items="listCount" :on-page-change="searchTutorList" />
  <TutorModal v-if="modalState.type === 'tutorDetail' && modalState.isOpen" :tutor-id="tutorId" />
</template>
<style>
@import './styled.css';
</style>
