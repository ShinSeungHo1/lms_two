<script setup>
import router from '@/router';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';

const lectureTag = ref('lecName');
const lectureTitle = ref('');
const startDate = ref('');
const endDate = ref('');

const searchLecture = () => {
  if (startDate.value > endDate.value) {
    ElMessage.error('잘못된 강의기간 조회입니다. 조회 날짜를 확인하여 주세요');
    startDate.value = '';
    endDate.value = '';
    return;
  }

  const query = [];

  !lectureTag.value || query.push(`searchTag=${lectureTag.value}`);
  !lectureTitle.value || query.push(`searchTitle=${lectureTitle.value}`);
  !startDate.value || query.push(`searchStDate=${startDate.value}`);
  !endDate.value || query.push(`searchEdDate=${endDate.value}`);

  const queryString = query.length > 0 ? `?${query.join('&')}` : '';

  router.push(queryString);
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>
<template>
  <div class="lecture-search-container">
    <div class="lecture-search-selector-input">
      <select id="lecture-search-selector" v-model="lectureTag">
        <option class="lecture-search-selector-option" value="lecName">강의명</option>
        <option class="lecture-search-selector-option" value="lecInstructorName">강사명</option>
        <option class="lecture-search-selector-option" value="lecRoomName">강의실</option>
      </select>
    </div>
    <div class="lecture-search-title-input">
      <input v-model.lazy="lectureTitle" class="lecture-search-title" type="text" />
    </div>
    <div class="lecture-search-date-input">
      <input v-model="startDate" class="lecture-search-date" type="date" />
      <span class="between-lecture-date">~</span>
      <input v-model="endDate" class="lecture-search-date" type="date" />
    </div>
    <div class="lecture-search-btn-container">
      <button id="lecture-search-btn" @click="searchLecture">검색</button>
    </div>
  </div>
</template>
<style>
@import './styled.css';
</style>
