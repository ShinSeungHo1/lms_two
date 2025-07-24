<script setup>
import router from '@/router';
import { useModalState } from '@/stores/modalState';
import { onMounted, ref } from 'vue';
import { useUserInfo } from '@/stores/loginInfoState';

const { user } = useUserInfo();

const modalState = useModalState();
const selectType = ref('title');
const inputText = ref('');

const handlerSearch = () => {
  const query = [];
  if (selectType.value === 'title') !selectType.value || query.push(`title=${inputText.value}`);
  else if (selectType.value === 'writer')
    !selectType.value || query.push(`writer=${inputText.value}`);

  const queryString = query.length > 0 ? `?${query}` : '';
  router.push(queryString);
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>

<template>
  <div class="qna-container">
    <div class="input-box">
      <select v-model="selectType">
        <option value="title">제목</option>
        <option value="writer">작성자</option>
      </select>
      <input v-model="inputText" type="text" />
      <button @click="handlerSearch">검색</button>
      <button
        v-if="user.userType === 'S'"
        @click="modalState.$patch({ isOpen: true, type: 'create' })"
      >
        신규
      </button>
    </div>
  </div>
</template>

<style>
@import './styled.css';
</style>
