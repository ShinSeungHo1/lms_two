<script setup>
import axios from 'axios';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';
import { useModalState } from '@/stores/modalState';

import PageNavigation from '@/components/common/PageNavigation.vue';
import RecruitModal from '../RecruitModal/RecruitModal.vue';

const headList = ['로그인ID', '학생명', '입사일', '퇴사일', '업체명', '재직여부'];

const route = useRoute();
const modalState = useModalState();

const recruitList = ref([]);
const listCount = ref(0);

const loginId = ref('');

const recruitDetailModal = (id) => {
  modalState.$patch({ isOpen: true, type: 'recruitDetail' });
  loginId.value = id;
};

const searchRecruitList = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/recruitListBody.do', urlParam)
    .then((res) => {
      console.log(res, '!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!');
      console.log('취업 리스트 불러오기 성공!');
      recruitList.value = res.data.list;
      listCount.value = res.data.count;
    })
    .catch((err) => {
      ElMessage.error('서버 오류 발생');
      console.log('취업 리스트 서버 오류 발생 err : ', err);
    });
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

onMounted(() => {
  searchRecruitList();
});
</script>
<template>
  <div class="recruit-table-container">
    <table class="recruit-table">
      <thead class="recruit-table-head">
        <tr class="recruit-table-head-row">
          <th v-for="(head, index) in headList" :key="`h-${index}`" class="recruit-table-head-cell">
            {{ head }}
          </th>
        </tr>
      </thead>
      <tbody class="recruit-table-body">
        <tr v-for="recruit in recruitList" :key="recruit.empId" class="recruit-table-body-row">
          <td class="recruit-table-body-cell">
            {{ recruit.loginID }}
          </td>
          <td class="recruit-table-body-cell" @click="recruitDetailModal(recruit.loginID)">
            {{ recruit.studentName }}
          </td>
          <td class="recruit-table-body-cell">
            {{ formatDate(recruit.empJoinDate) }}
          </td>
          <td class="recruit-table-body-cell">
            {{ formatDate(recruit.empRetireDate) }}
          </td>
          <td class="recruit-table-body-cell">
            {{ recruit.empName }}
          </td>
          <td class="recruit-table-body-cell">
            {{ recruit.studentsEmpStatus === 'Y' ? '취업' : '실업' }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation
    :items-per-page="5"
    :total-items="listCount"
    :on-page-change="searchRecruitList"
  />
  <RecruitModal
    v-if="modalState.type === 'recruitDetail' && modalState.isOpen"
    :login-id="loginId"
    @success="searchRecruitList"
  />
</template>

<style>
@import './styled.css';
</style>
