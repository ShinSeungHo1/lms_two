<script setup></script>

<template>
  <div class="company-data">
    <div class="input-box">
      기업이름:
      <input type="text" class="input-company-name" />
    </div>
    <div class="date-box">
      가입일자:
      <input type="date" class="input-company-date" />
      ~
      <input type="date" class="input=company-date" />
    </div>

    <button id="search-company">검색</button>
    <button id="create-company">신규</button>
  </div>
</template>

<style>
@import './styled.css';
</style>
