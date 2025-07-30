<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import ListModal from '../ListModal/ListModal.vue';

import axios from 'axios';
import { useModalState } from '@/stores/modalState';
import { ElMessage } from 'element-plus';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();
const modalState = useModalState();

const headList = ['강의명', '강사', '강의 시작일', '강의 종료일', '정원', '신청인원', '강의실'];
const lectureList = ref([]);
const listCount = ref(0);
const lecId = ref(0);

const serarchLectureList = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);
  axios
    .post('/api/lecture/lectureListBody.do', urlParam)
    .then((res) => {
      console.log(res);
      console.log('강의 목록 불러오기 성공');
      lectureList.value = res.data.list;
      listCount.value = res.data.count;
    })
    .catch((err) => {
      ElMessage.error('서버 오류 발생');
      console.log('강의 목록 불러오기 서버 오류 발생 err : ', err);
    });
};

const openLecModal = (id) => {
  modalState.$patch({ isOpen: true, type: 'lecDetail' });
  lecId.value = id;
};

watch(
  () => route.query,
  () => {
    serarchLectureList();
  },
);

onMounted(() => {
  serarchLectureList();
});
</script>
<template>
  <div class="lecture-table-container">
    <table class="lecture-table">
      <thead class="lecture-table-head">
        <tr class="lecture-table-head-row">
          <th v-for="(head, index) in headList" :key="`h-${index}`" class="lecture-table-head-cell">
            {{ head }}
          </th>
        </tr>
      </thead>
      <tbody class="lecture-table-body">
        <tr v-for="lecture in lectureList" :key="lecture.lecId" class="lecture-table-body-row">
          <th class="lecture-table-body-cell" @click="openLecModal(lecture.lecId)">
            {{ lecture.lecName }}
          </th>
          <th class="lecture-table-body-cell">{{ lecture.lecInstructorName }}</th>
          <th class="lecture-table-body-cell">{{ lecture.lecStartDate?.substring(0, 10) }}</th>
          <th class="lecture-table-body-cell">{{ lecture.lecEndDate?.substring(0, 10) }}</th>
          <th class="lecture-table-body-cell">{{ lecture.lecPersonnel }}</th>
          <th class="lecture-table-body-cell">{{ lecture.courseCntPersonnel }}</th>
          <th class="lecture-table-body-cell">{{ lecture.lecRoomName }}</th>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation
    :items-per-page="5"
    :total-items="listCount"
    :on-page-change="serarchLectureList"
  />
  <ListModal
    v-if="modalState.isOpen && modalState.type === 'lecDetail'"
    :lec-id="lecId"
    @success="serarchLectureList"
  />
</template>
<style>
@import './styled.css';
</style>
