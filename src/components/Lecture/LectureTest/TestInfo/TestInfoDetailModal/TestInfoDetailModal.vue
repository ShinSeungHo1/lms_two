<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';

const props = defineProps({
  testId: {
    type: Number,
    default: 0,
  },
  lecId: {
    type: Number,
    default: 0,
  },
});

const headList = ['번호', '내용', '배점', '정답', '비고'];

const testInfoList = ref([]);


const searchTestInfoDetail = () => {
  const param = {
    testId: props.testId,
    lecId: props.lecId,
  };

  const urlParam = new URLSearchParams(param);

  axios.post('/api/lecture/testInfoDetail.do', urlParam).then((res) => {
    console.log(res);
    testInfoList.value = res.data;
  });
};

onMounted(() => {
  searchTestInfoDetail();
});
</script>
<template>
  <div class="test-info-detail-container">
    <div class="test-info-detail-title-container">
      <div class="test-info-detail-title">시험 상세</div>
      <button id="close-x-btn" @click="closeModal">x</button>
    </div>
    <hr />
    <div class="test-info-detail-table-title">문제 목록</div>
    <div class="test-info-detail-table-container">
      <table class="test-info-detail-table">
        <thead class="test-info-detail-table-head">
          <tr class="test-info-detail-table-head-row">
            <th
              v-for="(head, index) in headList"
              :key="`h-${index}`"
              class="test-info-detail-table-head-cell"
            >
              {{ head }}
            </th>
          </tr>
        </thead>
        <tbody class="test-info-detail-table-body">
          <tr class="test-info-detail-table-body-row">
            <th class="test-info-detail-table-body-cell"></th>
            <th class="test-info-detail-table-body-cell"></th>
            <th class="test-info-detail-table-body-cell"></th>
            <th class="test-info-detail-table-body-cell"></th>
            <th class="test-info-detail-table-body-cell"></th>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
