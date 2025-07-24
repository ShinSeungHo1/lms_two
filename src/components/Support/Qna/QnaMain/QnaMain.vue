<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import QnaModal from '../QnaModal/QnaModal.vue';
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';

const modalState = useModalState();

const qnaId = ref(0);

const qnaList = ref([]);
const qnaCount = ref(0);
const route = useRoute();

const qnaSearch = (cPage = 1) => {
  const params = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(params);
  axios
    .post('/api/support/getQnaListBody.do', urlParam)
    .then((res) => {
      console.log(res);
      qnaList.value = res.data.list;
      qnaCount.value = res.data.count;
    })
    .catch((err) => {
      console.log('QnA List 불러오기 실패 : ' + err);
    });
};

const updateQna = (id) => {
  modalState.$patch({ isOpen: true, type: 'update' });
  qnaId.value = id;
};

watch(
  () => route.query,
  () => {
    qnaSearch();
  },
);
onMounted(() => {
  qnaSearch();
});
</script>

<template>
  <div class="qna-main-container">
    <table class="qna-table">
      <thead class="qna-table-header">
        <tr>
          <th>No.</th>
          <th>강의명</th>
          <th>제목</th>
          <th>작성일</th>
          <th>작성자</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="qnaCount > 0">
          <tr v-for="qna in qnaList" :key="qna.qnaId" class="qna-table-row">
            <td class="qna-cell">{{ qna.qnaId }}</td>
            <td class="qna-cell">{{ qna.lecName }}</td>
            <td class="qna-cell cursor-pointer hover:underline" @click="updateQna(qna.qnaId)">
              {{ qna.qnaTitle }}
            </td>
            <td class="qna-cell">{{ qna.qnaAnswerDate }}</td>
            <td class="qna-cell">{{ qna.qnaAnswerWriter }}</td>
          </tr>
        </template>
        <template v-else>
          <tr>
            <td colspan="4" class="qna-empty-row">일치하는 QnA가 없습니다.</td>
          </tr>
        </template>
      </tbody>
    </table>
    <PageNavigation :total-items="qnaCount" :items-per-page="5" :on-page-change="qnaSearch" />
  </div>
  <QnaModal
    v-if="modalState.type === 'update' && modalState.isOpen"
    :qna-id="qnaId"
    @delete-success="qnaSearch"
    @save-success="qnaSearch"
  />
  <QnaModal
    v-if="modalState.type === 'create' && modalState.isOpen"
    @delete-success="qnaSearch"
    @save-success="qnaSearch"
    @update-success="qnaSearch"
    @answer-fail="qnaSearch"
  />
</template>

<style>
@import './styled.css';
</style>
