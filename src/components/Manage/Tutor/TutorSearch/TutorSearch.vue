<script setup>
import router from '@/router';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';

const tutorName = ref('');
const tutorStatus = ref('');
const regStDate = ref('');
const regEdDate = ref('');

const searchTutor = () => {
  const today = new Date().toISOString().split('T')[0];

  if (regStDate.value && regStDate.value > today) {
    ElMessage.error('시작일은 오늘 이후 날짜일 수 없습니다.');
    return;
  }

  if (regEdDate.value && regEdDate.value < today) {
    ElMessage.error('종료일은 오늘 이후 날짜일 수 없습니다.');
    return;
  }

  if (regStDate.value && regEdDate.value && regStDate.value > regEdDate.value) {
    ElMessage.error('종료일은 시작일보다 빠를 수 없습니다.');
    return;
  }

  const query = [];

  !tutorName.value || query.push(`searchName=${tutorName.value}`);
  !tutorStatus.value || query.push(`searchStatusYn=${tutorStatus.value}`);
  !regStDate.value || query.push(`regStDate=${regStDate.value}`);
  !regEdDate.value || query.push(`regEdDate=${regEdDate.value}`);

  const queryString = query.length > 0 ? `?${query.join('&')}` : '';
  router.push(queryString);
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>
<template>
  <div class="tutor-search-container">
    <div class="tutor-name-container">
      <input v-model.lazy="tutorName" class="tutor-name-input" type="text" />
    </div>
    <div class="tutor-status-container">
      <select id="tutor-status-selector" v-model="tutorStatus">
        <option class="tutor-status-option" value="">선택</option>
        <option class="tutor-status-option" value="W">승인대기</option>
        <option class="tutor-status-option" value="Y">재직</option>
        <option class="tutor-status-option" value="N">퇴직</option>
      </select>
    </div>
    <div class="tutor-join-date-conatiner">
      <div class="date-inputs">
        <input v-model="regStDate" class="tutor-join-stDate-input" type="date" />
        <span class="date-separator">~</span>
        <input v-model="regEdDate" class="tutor-join-edDate-input" type="date" />
      </div>
    </div>
    <button class="tutor-search-btn" @click="searchTutor">검색</button>
  </div>
</template>
<style>
@import './styled.css';
</style>
