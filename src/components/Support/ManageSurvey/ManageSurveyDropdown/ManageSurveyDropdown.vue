<script setup>
import { ref } from 'vue';
import ManageSurveyTable from '../ManageSurveyTable/ManageSurveyTable.vue';

const isDropdownOpen = ref(false);

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
};

const textOfSelectedItem = ref('');
const selectedManageItem = ref('');
const manageItemList = [
  {
    value: 'completedSurvey',
    text: '완료 목록 조회',
  },
  {
    value: 'getResult',
    text: '결과 조회',
  },
];

const selectManageItem = (manageItem) => {
  selectedManageItem.value = manageItem.value;
  textOfSelectedItem.value = manageItem.text;
  isDropdownOpen.value = false;
};
</script>

<template>
  <div class="form-group">
    <label class="form-label required">관리 항목</label>
    <div class="dropdown-container">
      <input
        :value="textOfSelectedItem || ''"
        type="text"
        class="form-input cursor-pointer"
        placeholder="클릭해서 항목 선택"
        readonly
        @click="toggleDropdown"
      />
      <!-- 강의 목록 드롭다운 -->
      <div v-if="isDropdownOpen" class="lecture-dropdown-list">
        <div
          v-for="(manageItem, index) in manageItemList"
          :key="index"
          class="lecture-dropdown-item"
          @click="selectManageItem(manageItem)"
        >
          {{ manageItem.text }}
        </div>
      </div>
    </div>
  </div>
  <ManageSurveyTable :manage-item="selectedManageItem" />
</template>

<style>
@import './styled.css';
</style>
