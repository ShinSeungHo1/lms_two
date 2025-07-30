<script setup>
import router from '@/router';
import { useUserInfo } from '@/stores/loginInfoState';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';

const { user } = useUserInfo();

const testTag = ref('lecName');
const testTitle = ref('');
const startDate = ref('');
const endDate = ref('');

const searchTestInfo = () => {
  if (startDate.value > endDate.value) {
    ElMessage.error('잘못된 시험일자 조회입니다. 조회 날짜를 확인해 주세요');
    startDate.value = '';
    endDate.value = '';
    return;
  }
  const query = [];

  !testTag.value || query.push(`searchTage=${testTag.value}`);
  !testTitle.value || query.push(`searchTitle=${testTitle.value}`);
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
  <div class="test-search-container">
    <div class="test-search-content">
      <div class="test-search-tag">
        <select id="test-search-tag-selector" v-model="testTag">
          <option class="test-search-tag-option" value="lecName">강의명</option>
          <option class="test-search-tag-option" value="lecInstructorName">강사명</option>
          <option class="test-search-tag-option" value="lecRoomName">강의실</option>
        </select>
      </div>
      <div class="test-search-title-container">
        <input v-model.lazy="testTitle" class="test-search-title" type="text" />
      </div>
      <div class="test-search-date-container">
        <label class="test-search-date-label">강의 기간</label>
        <input v-model="startDate" class="test-search-date" type="date" />
        <span class="between-search-date"> ~ </span>
        <input v-model="endDate" class="test-search-date" type="date" />
      </div>
      <div class="test-search-btn-container">
        <button id="test-search-btn" @click="searchTestInfo">검색</button>
        <button v-if="user.userType === 'M'" id="test-create-btn">신규</button>
      </div>
    </div>
  </div>
</template>
<style>
@import './styled.css';
</style>
