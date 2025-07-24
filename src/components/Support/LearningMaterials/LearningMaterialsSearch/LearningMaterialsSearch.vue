<script setup>
import router from '@/router';
import { onMounted, ref } from 'vue';
import { useUserInfo } from '@/stores/loginInfoState';
import { useModalState } from '@/stores/modalState';

const { user } = useUserInfo();
const modalState = useModalState();

const select = ref('title');
const inputText = ref('');

const handlerSearch = () => {
  const query = [];

  !select.value ||
    query.push(select.value === 'title' ? `title=${inputText.value}` : `writer=${inputText.value}`);
  const queryString = query.length > 0 ? `?${query}` : '';

  router.push(queryString);
};

const createMtrModal = () => {
  modalState.$patch({ isOpen: true, type: 'createMtr' });
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>

<template>
  <div class="learningMaterials-container">
    <div class="input-box">
      <select v-model="select">
        <option value="title">제목</option>
        <option value="writer">작성자</option>
      </select>
      <input v-model.lazy="inputText" type="text" />
      <button @click="handlerSearch">검색</button>
      <button v-if="user.userType === 'T'" @click="createMtrModal">신규</button>
    </div>
  </div>
</template>

<style>
@import './styled.css';
</style>
