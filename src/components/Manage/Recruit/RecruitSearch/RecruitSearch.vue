<script setup>
import router from '@/router';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';

const studentName = ref('');
const searchStDate = ref('');
const searchEdDate = ref('');

const searchRecruit = () => {
  const startDate = searchStDate.value;
  const endDate = searchEdDate.value;

  if (startDate && endDate && startDate > endDate) {
    ElMessage.error('종료일은 시작일보다 빠를 수 없습니다.');
    searchEdDate.value = '';
    searchStDate.value = '';
    return;
  }

  const query = [];
  !studentName.value || query.push(`studentName=${studentName.value}`);
  !searchStDate.value || query.push(`searchStDate=${searchStDate.value}`);
  !searchEdDate.value || query.push(`searchEdDate=${searchEdDate.value}`);

  const queryString = query.length > 0 ? `?${query.join('&')}` : '';

  router.push(queryString);
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>
<template>
  <div class="recruit-search-container">
    <div class="recruit-search-title-input">
      <input id="recruit-search-title" v-model.lazy="studentName" type="text" />
    </div>
    <div class="recruit-search-date-input">
      <input v-model="searchStDate" class="recruit-search-date" type="date" />
      <span class="recruit-date-between">~</span>
      <input v-model="searchEdDate" class="recruit-search-date" type="date" />
    </div>
    <div class="recruit-search-btn-container">
      <button id="recruit-search-btn" @click="searchRecruit"></button>
    </div>
  </div>
</template>
<style>
@import './styled.css';
</style>
