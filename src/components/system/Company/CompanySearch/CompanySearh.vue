<script setup>
import router from '@/router';
import { useModalState } from '@/stores/modalState';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';

const modalState = useModalState();

const searchTitle = ref('');
const searchStDate = ref('');
const searchEdDate = ref('');

const handlerSearch = () => {
  if (new Date(searchStDate.value) > new Date(searchEdDate.value)) {
    ElMessage.error('조회 날짜를 다시 확인해 주세요');
    searchStDate.value = '';
    searchEdDate.value = '';
    return;
  }
  const query = [];

  !searchTitle.value || query.push(`companyName=${searchTitle.value}`);
  !searchStDate.value || query.push(`searchStDate=${searchStDate.value}`);
  !searchEdDate.value || query.push(`searchEdDate=${searchEdDate.value}`);

  const queryString = query.length > 0 ? `?${query.join('&')}` : '';

  router.push(queryString);
};

const handlerCreate = () => {
  modalState.$patch({ isOpen: true, type: 'createCompany' });
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>

<template>
  <div class="company-data">
    <div class="input-box">
      기업이름:
      <input v-model.lazy="searchTitle" type="text" class="input-company-name" />
    </div>
    <div class="date-box">
      가입일자:
      <input v-model="searchStDate" type="date" class="input-company-date" />
      ~
      <input v-model="searchEdDate" type="date" class="input-company-date" />
    </div>

    <button id="search-company" @click="handlerSearch">검색</button>
    <button id="create-company" @click="handlerCreate">신규</button>
  </div>
</template>

<style>
@import './styled.css';
</style>
