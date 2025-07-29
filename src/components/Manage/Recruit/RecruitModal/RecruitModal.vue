<script setup>
import axios from 'axios';
import { ElMessage, ElMessageBox } from 'element-plus';
import { onMounted, ref, watch } from 'vue';

import PageNavigation from '@/components/common/PageNavigation.vue';
import RecruitUCModal from '../RecruitUCModal/RecruitUCModal.vue';
import { useModalStateSub } from '@/stores/modalState';

const emits = defineEmits(['success']);

const props = defineProps({
  loginId: {
    type: String,
    default: '',
  },
});

const headList = ['번호', '학생명', '입사일', '퇴사일', '업체명', '재직여부', '-'];

const modalStateSub = useModalStateSub();

const recruitDetailList = ref([]);
const detailListCount = ref(0);
const empId = ref(0);

const pageKey = ref(0);

const searchRecruitDetail = (cPage = 1) => {
  const param = {
    loginID: props.loginId,
    currentPage: cPage,
    pageSize: 5,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/RecruitDetailBody.do', urlParam)
    .then((res) => {
      console.log('취업 세부사항 불러오기 성공!');
      console.log(res);
      recruitDetailList.value = res.data.list;
      detailListCount.value = res.data.count;
      recruitDetailList.value.forEach((item) => {
        console.log(item.empRetireDate, item.studentsEmpStatus);
        item.disableRetireBtn = false;
        if (item.empRetireDate || item.studentsEmpStatus === 'N') {
          item.disableRetireBtn = true;
          item.studentsEmpStatus = 'N';
        }
      });
    })
    .catch((err) => {
      ElMessage.error('서버 오류 발생');
      console.log('취업 세부사항 서버 오류 발생 err : ', err);
    });
};

const changeToRetire = (id) => {
  ElMessageBox.confirm('정말 퇴직하시겠습니까?', {
    confirmButtonText: '확인',
    cancelButtonText: '취소',
    type: 'info',
  })
    .then(() => {
      const param = {
        loginID: id,
      };

      const urlParam = new URLSearchParams(param);

      axios
        .post('/api/manage/RetireStudent.do', urlParam)
        .then((res) => {
          if (res.data.result === 'success') {
            ElMessage.success('퇴직 되었습니다.');
            searchRecruitDetail();
            emits('success');
          }
        })
        .catch((err) => {
          console.log('퇴직 변경 서버 오류 발생 err : ', err);
          ElMessage.error('서버 오류 발생');
        });
    })
    .catch(() => {
      ElMessage.info('퇴직 변경 취소');
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

const openUpdateModal = (id) => {
  modalStateSub.$patch({ isOpen: true, type: 'updateRecruit' });
  empId.value = id;
};

const openCreateModal = () => {
  modalStateSub.$patch({ isOpen: true, type: 'createRecruit' });
};

watch(
  () => props.loginId,
  () => {
    pageKey.value += 1;
    searchRecruitDetail();
  },
);

onMounted(() => {
  searchRecruitDetail();
});
</script>
<template>
  <div class="recruit-create-btn-container">
    <button id="recruit-create-btn" @click="openCreateModal">신규</button>
  </div>
  <div class="recruit-detail-table-container">
    <table class="recruit-detail-table">
      <thead class="recruit-detail-table-head">
        <tr class="recruit-detail-table-head-row">
          <th
            v-for="(head, index) in headList"
            :key="`h-${index}`"
            class="recruit-detail-table-head-cell"
          >
            {{ head }}
          </th>
        </tr>
      </thead>
      <tbody class="recruit-detail-table-body">
        <tr
          v-for="recruit in recruitDetailList"
          :key="recruit.empId"
          class="recruit-detail-table-body-row"
        >
          <td class="recruit-detail-table-body-cell">{{ recruit.empId }}</td>
          <td class="recruit-detail-table-body-cell" @click="openUpdateModal(recruit.empId)">
            {{ recruit.studentName }}
          </td>
          <td class="recruit-detail-table-body-cell">{{ formatDate(recruit.empJoinDate) }}</td>
          <td class="recruit-detail-table-body-cell">{{ formatDate(recruit.empRetireDate) }}</td>
          <td class="recruit-detail-table-body-cell">{{ recruit.empName }}</td>
          <td class="recruit-detail-table-body-cell">
            {{ recruit.studentsEmpStatus === 'Y' ? '취업' : '퇴직' }}
          </td>
          <td class="recruit-detail-table-body-cell">
            <button
              id="change-status-btn"
              :disabled="recruit.disableRetireBtn"
              @click="changeToRetire(props.loginId)"
            >
              퇴직
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation
    :key="pageKey"
    :items-per-page="5"
    :total-items="detailListCount"
    :on-page-change="searchRecruitDetail"
  />
  <RecruitUCModal
    v-if="modalStateSub.type === 'updateRecruit' && modalStateSub.isOpen"
    :emp-id="empId"
    @success="searchRecruitDetail"
  />
  <RecruitUCModal
    v-if="modalStateSub.type === 'createRecruit' && modalStateSub.isOpen"
    :login-id="props.loginId"
    @success="searchRecruitDetail"
  />
</template>
<style>
@import './styled.css';
</style>
