<script setup>
import router from '@/router';
import { onMounted, ref } from 'vue';

const commonGroup = ref('');
const groupCode = ref('');
const serialCode = ref('');

const handlerSearch = () => {
  const query = [];

  !commonGroup.value || query.push(`commonGroup=${commonGroup.value}`);
  !groupCode.value || query.push(`groupCode=${groupCode.value}`);
  !serialCode.value || query.push(`detailCode=${serialCode.value}`);

  const queryString = query.length > 0 ? `?${query.join('&')}` : '';
  router.push(queryString);
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>
<template>
  <div class="search-container">
    <div class="search-classified">
      분류:
      <select id="classified" v-model="commonGroup">
        <option value="">분류란을 선택하세요</option>
        <option value="equip">장비</option>
        <option value="loc">지역</option>
      </select>
    </div>
    <div class="search-group-code">
      그룹코드:
      <select id="group-code" v-model="groupCode">
        <option value="">그룹코드를 선택하세요</option>
        <option value="com">컴퓨터</option>
        <option value="ms">마우스</option>
        <option value="msp">마우스패드</option>
        <option value="kb">키보드</option>
        <option value="mt">모니터</option>
        <option value="dk">책사</option>
        <option value="chr">의자</option>
        <option value="wb">화이트보드</option>
        <option value="etc">기타</option>
      </select>
    </div>
    <div class="serial-code">
      시리얼코드:
      <input v-model.lazy="serialCode" type="text" />
    </div>
    <button @click="handlerSearch">검색</button>
  </div>
</template>
<style>
@import './styled.css';
</style>
