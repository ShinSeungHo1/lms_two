<script setup>
import ContentBox from '@/components/common/ContentBox.vue';
import TestSearch from '@/components/Lecture/LectureTest/TestCommon/TestSearch/TestSearch.vue';
import Test from '@/components/Lecture/LectureTest/TestVerStd/TestVerStdMain/Test.vue';
import { useUserInfo } from '@/stores/loginInfoState';

const { user } = useUserInfo();
</script>
<template>
  <ContentBox>시험 정보</ContentBox>
  <TestSearch />
  <Test v-if="user.userType === 'S'" />
</template>
