<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';
import LectureSurveyQuestion from '../LectureSurveyQuestion/LectureSurveyQuestion.vue';

const isDropdownOpen = ref(false);

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
};

const selectedLecId = ref(0);
const selectedLecture = ref({});
const lectureList = ref([]);

const lectureListByLoginId = () => {
  axios.get('/api/support/lecture-surveyJson').then((res) => {
    console.log('LoginID에 맞는 Lecture 정보 불러오기 성공', res);
    lectureList.value = res.data.lectures;
    console.log(lectureList.value);
  });
};

const selectLecture = (lecture) => {
  selectedLecture.value = lecture;
  selectedLecId.value = lecture.lecId;
  isDropdownOpen.value = false;
};

onMounted(() => {
  lectureListByLoginId();
});
</script>

<template>
  <div class="form-group">
    <label class="form-label required">강의명</label>
    <div class="dropdown-container">
      <input
        :value="selectedLecture.lecName || ''"
        type="text"
        class="form-input cursor-pointer"
        placeholder="강의명을 선택해주세요"
        readonly
        @click="toggleDropdown"
      />
      <!-- 강의 목록 드롭다운 -->
      <div v-if="isDropdownOpen" class="lecture-dropdown-list">
        <div
          v-for="lecture in lectureList"
          :key="lecture.lecId"
          class="lecture-dropdown-item"
          @click="selectLecture(lecture)"
        >
          {{ lecture.lecName }}
        </div>
        <div v-if="lectureList.length === 0" class="lecture-dropdown-empty">강의가 없습니다.</div>
      </div>
    </div>
  </div>
  <LectureSurveyQuestion :lec-id="selectedLecId" />
</template>

<style>
@import './styled.css';
</style>
