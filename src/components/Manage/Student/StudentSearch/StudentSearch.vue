<script setup>
import router from '@/router';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';

const studentName = ref('');
const studentStatus = ref('');
const studentRegStDate = ref('');
const studentRegEdDate = ref('');

const searchStudent = () => {
  const today = new Date().toISOString().split('T')[0];

  if (studentRegStDate.value && studentRegStDate.value > today) {
    ElMessage.error('시작일은 오늘 이후 날짜일 수 없습니다.');
    return;
  }

  if (studentRegEdDate.value && studentRegEdDate.value < today) {
    ElMessage.error('종료일은 오늘 이후 날짜일 수 없스니다.');
    return;
  }

  if (
    studentRegStDate.value &&
    studentRegEdDate.value &&
    studentRegStDate.value > studentRegEdDate.value
  ) {
    ElMessage.error('종료일은 시작일보다 빠를 수 없습니다.');
    return;
  }

  const query = [];

  !studentName.value || query.push(`searchName=${studentName.value}`);
  !studentStatus.value || query.push(`searchStatusYn=${studentStatus.value}`);
  !studentRegStDate.value || query.push(`regStDate=${studentRegStDate.value}`);
  !studentRegEdDate.value || query.push(`regEdDate=${studentRegEdDate.value}`);

  const queryString = query.length > 0 ? `?${query.join('&')}` : '';

  router.push(queryString);
};

onMounted(() => {
  window.location.search && router.replace(window.location.pathname);
});
</script>

<template>
  <div class="search-container">
    <div class="student-name-input">
      <input id="student-name" v-model.lazy="studentName" type="text" />
    </div>
    <div class="student-status-input">
      <select id="student-status-selector" v-model="studentStatus">
        <option value="">선택</option>
        <option value="W">승인대기</option>
        <option value="Y">재학</option>
        <option value="N">탈퇴</option>
      </select>
    </div>
    <div class="student-regDate-input">
      가입 기간:
      <input id="student-regStDate" v-model="studentRegStDate" type="date" />
      ~
      <input id="student-regEdDate" v-model="studentRegEdDate" type="date" />
    </div>
    <div class="search-btn-container">
      <button id="search-btn" @click="searchStudent">검색</button>
    </div>
  </div>
</template>

<!-- style scoped는 해당 컴포넌트에서만 스타일을 적용시키는 방법이다.
    그러나, scoped는 컴파일 시점에 고유한 속성을 컴포넌트 요소에 추가하는데,
    외부 css 파일은 이 고유 속성을 알지 못하므로 스타일이 적용안된다.

    scoped을 사용하지 않고, css파일을 import 하거나, <style scoped> 안에 css를 구성해야한다.
-->
<style>
@import './styled.css';
</style>
