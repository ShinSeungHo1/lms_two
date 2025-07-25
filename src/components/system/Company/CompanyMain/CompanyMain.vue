<script setup>
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { useModalState } from '@/stores/modalState';

import PageNavigation from '@/components/common/PageNavigation.vue';
import CompanyModal from '../CompanyModal/CompanyModal.vue';

const pageKey = ref(0);

const headerList = ['번호', '회사명', '대표명', '휴대전화', '주소', '이메일', '가입일자'];
const route = useRoute();
const modalState = useModalState();

const companyList = ref([]);
const companyCount = ref(0);

const companyId = ref(0);

const emitSuccess = () => {
  pageKey.value += 1;

  searchCompanyList();
};

const searchCompanyList = (cPage = 1) => {
  const param = {
    currentPage: cPage,
    pageSize: 5,
    ...route.query,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/system/companyListBody.do', urlParam)
    .then((res) => {
      console.log(res);
      console.log('회사 정보 불러오기 성공!');
      companyList.value = res.data.list;
      companyCount.value = res.data.count;
    })
    .catch(() => {
      console.log('회사 정보 불러오기 실패!');
    });
};

const showCompanyDetailModal = (id) => {
  modalState.$patch({ isOpen: true, type: 'detailCompany' });
  companyId.value = id;
};

watch(
  () => route.query,
  () => {
    searchCompanyList();
  },
);

onMounted(() => {
  searchCompanyList();
});
</script>

<template>
  <div class="table-company-container">
    <table class="table-company">
      <thead class="table-company-header">
        <tr class="table-company-header-row">
          <th
            v-for="(header, index) in headerList"
            :key="`h-${index}`"
            class="table-company-header-cell"
          >
            {{ header }}
          </th>
        </tr>
      </thead>
      <tbody class="table-company-body">
        <tr v-for="company in companyList" :key="company.companyId" class="table-company-body-row">
          <td class="table-company-body-cell">{{ company.companyId }}</td>
          <td class="table-company-body-cell" @click="showCompanyDetailModal(company.companyId)">
            {{ company.companyName }}
          </td>
          <td class="table-company-body-cell">{{ company.companyCeo }}</td>
          <td class="table-company-body-cell">{{ company.companyHp }}</td>
          <td class="table-company-body-cell">{{ company.companyLoc }}</td>
          <td class="table-company-body-cell">{{ company.companyEmail }}</td>
          <td class="table-company-body-cell">
            {{ new Date(company.companyRegDate).toLocaleDateString() }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <PageNavigation
    :key="pageKey"
    :items-per-page="5"
    :total-items="companyCount"
    :on-page-change="searchCompanyList"
  />
  <CompanyModal
    v-if="modalState.type === 'detailCompany' && modalState.isOpen"
    :company-id="companyId"
    @success="emitSuccess"
  />
  <CompanyModal
    v-if="modalState.type === 'createCompany' && modalState.isOpen"
    @success="emitSuccess"
  />
</template>

<style>
@import './styled.css';
</style>
