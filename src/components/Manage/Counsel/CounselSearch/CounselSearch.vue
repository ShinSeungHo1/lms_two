<script setup>
import router from '@/router';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';

const counselTitle = ref('');
const counselStDate = ref('');
const counselEdDate = ref('');

const searchCounsel = () => {
  const today = new Date().toISOString().split('T')[0];

  if (counselStDate.value && counselStDate.value > today) {
    ElMessage.error('시작일은 오늘 이후 날짜일 수 없습니다.');
    return;
  }

  if (counselEdDate.value && counselEdDate.value < today) {
    ElMessage.error('종료일은 오늘 이후 날짜일 수 없습니다.');
    return;
  }

  if (counselStDate.value && counselEdDate.value && counselStDate.value > counselEdDate.value) {
    ElMessage.error('종료일은 시작일보다 빠를 수 없습니다.');
    return;
  }

  const query = [];

  !counselTitle.value || query.push(`searchTitle=${counselTitle.value}`);
  !counselStDate.value || query.push(`startDate=${counselStDate.value}`);
  !counselEdDate.value || query.push(`endDate=${counselEdDate.value}`);

  const queryString = query.length > 0 ? `?${query.join('&')}` : '';

  router.push(queryString);
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>
<template>
  <div class="counsel-search-container">
    <div class="counsel-name-input">
      <input v-model.lazy="counselTitle" class="counsel-name" type="text" />
    </div>
    <div class="counsel-date-input">
      <input v-model="counselStDate" class="counsel-stDate" type="date" />
      <span class="counsel-between-date">~</span>
      <input v-model="counselEdDate" class="counsel-edDate" type="date" />
    </div>
    <div class="counsel-btn-container">
      <button class="counsel-search-btn" @click="searchCounsel"></button>
      <button class="counsel-create-btn" @click="createCounsel"></button>
    </div>
  </div>
</template>
<style>
@import './styled.css';
</style>
