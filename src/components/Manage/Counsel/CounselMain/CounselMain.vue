<script setup>
import axios from 'axios';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';

import PageNavigation from '@/components/common/PageNavigation.vue';

const headList = ['No.', '상담 과목', '상담 제목', '상담 학생', '상담일'];

const route = useRoute();

const counselList = ref([]);
const listCount = ref(0);
const lectureList = ref([]);

const searchCounsel = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/manage/getCounselListBody.do', urlParam)
    .then((res) => {
      console.log(res);
      console.log('상담 리스트 불러오기 성공');
      listCount.value = res.data.counselCnt;
      counselList.value = res.data.counselList;
      lectureList.value = res.data.lectures;
    })
    .catch((err) => {
      ElMessage.error('서버 오류 발생');
      console.log('상담 리스트 서버 오류 발생 err: ', err);
    });
};

const counselDetailModal = (id, lecId) => {};

onMounted(() => {
  searchCounsel();
});
</script>
<template>
  <div class="counsel-table-container">
    <table class="counsel-table">
      <thead class="counsel-table-head">
        <tr class="counsel-table-head-row">
          <th v-for="(head, index) in headList" :key="index" class="counsel-table-head-cell">
            {{ head }}
          </th>
        </tr>
      </thead>
      <tbody v-if="counselList.length > 0" class="counsel-table-body">
        <tr v-for="counsel in counselList" :key="counsel.counselId" class="counsel-table-body-row">
          <td class="counsel-table-body-cell">
            {{ counsel.counselId }}
          </td>
          <td class="counsel-table-body-cell">
            {{ counsel.lecName }}
          </td>
          <td
            class="counsel-table-body-cell"
            @click="counselDetailModal(counsel.counselId, counsel.lecId)"
          >
            {{ counsel.counselTitle }}
          </td>
          <td class="counsel-table-body-cell">
            {{ counsel.counselStudentName }}
          </td>
          <td class="counsel-table-body-cell">
            {{ counsel.counselRegDate?.substring(0, 10) }}
          </td>
        </tr>
      </tbody>
      <tbody v-else class="counsel-table-empty">
        <tr class="counsel-table-empty-row">
          <td class="counsel-table-empty-cell">상담 리스트가 없습니다.</td>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation :items-per-page="5" :total-items="listCount" :on-page-change="searchCounsel" />
</template>
<style>
@import './styled.css';
</style>
