<script setup>
import { useUserInfo } from '@/stores/loginInfoState';
import { useModalState } from '@/stores/modalState';
import { ElMessage } from 'element-plus';
import { onMounted, ref } from 'vue';
import axios from 'axios';

const props = defineProps({
  lecId: {
    type: Number,
    default: 0,
  },
});

const emits = defineEmits(['success']);

const modalState = useModalState();
const { user } = useUserInfo();

const lecDetail = ref({});
const lecWeekPlanList = ref([]);

const showApplicationBtn = ref(false);

const emitsAndCloseModal = (emit) => {
  modalState.$patch({ isOpen: false });
  emits(emit);
};

const searchLecDetail = () => {
  const param = {
    lecId: props.lecId,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/lecture/lectureDetail.do', urlParam)
    .then((res) => {
      console.log(res);
      console.log('강의 세부사항 불러오기 성공!');
      lecDetail.value = res.data.lectureDetailValue;

      const detail = lecDetail.value;

      lecWeekPlanList.value = detail.lecWeekPlanList;

      if (!detail.lecGoal?.trim()) {
        lecDetail.value.lecGoal = '아직 입력된 정보가 없습니다.';
      }

      if (!detail.lecContent?.trim()) {
        lecDetail.value.lecContent = '아직 입력된 정보가 없습니다.';
      }

      if (!detail.lecSpecifics?.trim()) {
        lecDetail.value.lecSpecifics = '아직 입력된 정보가 없습니다.';
      }

      if (user.userType === 'S' && res.data.isLectureRegistrationAvailable) {
        showApplicationBtn.value = true;
      }
    })
    .catch((err) => {
      console.log('강의 세부사항 불러오기 서버 오류 발생 err: ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const lectureRegister = () => {
  const param = {
    lecId: props.lecId,
    studentId: user.loginId,
  };

  const urlParam = new URLSearchParams(param);

  axios
    .post('/api/lecture/lectureStdRegister.do', urlParam)
    .then((res) => {
      if (res.data.result === 'success') {
        ElMessage.success('신청되었습니다.');
        emitsAndCloseModal('success');
      } else if (res.data.result === 'loginIdNotExist') {
        ElMessage.error(
          '학생정보가 등록되어 있지 않아 수강 신청할 수 없습니다. 관리자에게 문의하세요',
        );
      } else if (res.data.result === 'lecIdAlreadyExist') {
        ElMessage.error('이미 수강 신청한 강의입니다.');
      } else if (res.data.result === 'lecExceedsCapacity') {
        ElMessage.error('현재 수강 인원이 모두 마감되었습니다. 다른 강의를 확인해 주세요');
      } else {
        ElMessage.error('수강 신청 실패했습니다...');
      }
    })
    .catch((err) => {
      console.log('수강 신청 서버 오류 발생 err : ', err);
      ElMessage.error('서버 오류 발생');
    });
};

const closeModal = () => {
  emitsAndCloseModal('success');
};

onMounted(() => {
  searchLecDetail();
});
</script>
<template>
  <div class="lecture-modal-backdrop">
    <div class="lecture-modal-content">
      <div class="lecture-detail-container">
        <div class="lecture-detail-title">
          강의 상세 및 계획서
          <button id="close-x-btn" @click="closeModal">x</button>
        </div>
        <div class="lecture-detail-content">
          <div class="lecture-detail-content-title">강의 상세</div>
          <div class="lecture-detail-content-row">
            <div class="lecture-detail-field">강의</div>
            <div class="lecture-detail-info">{{ lecDetail.lecName }}</div>
          </div>
          <div class="lecture-detail-content-row">
            <div class="lecture-detail-content-divide">
              <div class="lecture-detail-field">강사</div>
              <div class="lecture-detail-info">{{ lecDetail.lecInstructorId }}</div>
            </div>
            <div class="lecture-detail-content-divide">
              <div class="lecture-detail-field">강의실</div>
              <div class="lecture-detail-info">{{ lecDetail.lecRoomName }}</div>
            </div>
          </div>
          <div class="lecture-detail-content-row">
            <div class="lecture-detail-content-divide">
              <div class="lecture-detail-field">이메일</div>
              <div class="lecture-detail-info">{{ lecDetail.insEmail }}</div>
            </div>
            <div class="lecture-detail-content-divide">
              <div class="lecture-detail-field">연락처</div>
              <div class="lecture-detail-info">{{ lecDetail.insHp }}</div>
            </div>
          </div>
          <div class="lecture-detail-content-row">
            <div class="lecture-detail-content-divide">
              <div class="lecture-detail-field">강의 시작일</div>
              <div class="lecture-detail-info">{{ lecDetail.lecStartDate }}</div>
            </div>
            <div class="lecture-detail-content-divide">
              <div class="lecture-detail-field">강의 종료일</div>
              <div class="lecture-detail-info">{{ lecDetail.lecEndDate }}</div>
            </div>
          </div>
          <div class="lecture-detail-content-row">
            <div class="lecture-detail-content-divide">
              <div class="lecture-detail-field">정원</div>
              <div class="lecture-detail-info">{{ lecDetail.lecPersonnel }}</div>
            </div>
            <div class="lecture-detail-content-divide">
              <div class="lecture-detail-field">강의 일수(주차)</div>
              <div class="lecture-detail-info">
                {{ `${lecDetail.lecDaysCnt}(${lecDetail.lecSectionCnt}주차)` }}
              </div>
            </div>
          </div>
        </div>
        <hr />
        <div class="lecture-plan-container">
          <div class="lecture-plan-title">강의 계획</div>
          <div class="lecture-plan-content-column">
            <div class="lecture-plan-field">강의목표</div>
            <div class="lecture-plan-info">{{ lecDetail.lecGoal }}</div>
          </div>
          <div class="lecture-plan-content-column">
            <div class="lecture-plan-field">강의내용</div>
            <div class="lecture-plan-info">{{ lecDetail.lecContent }}</div>
          </div>
          <div class="lecture-plan-content-column">
            <div class="lecture-plan-field">강의기타사항</div>
            <div class="lecture-plan-info">{{ lecDetail.lecSpecifics }}</div>
          </div>
          <table class="lecture-plan-table">
            <thead class="lecture-plan-table-head">
              <tr class="lecture-plan-table-head-row">
                <th class="lecture-plan-table-head-cell">주차</th>
                <th class="lecture-plan-table-head-cell">학습목표</th>
                <th class="lecture-plan-table-head-cell">학습내용</th>
              </tr>
            </thead>
            <tbody class="lecture-plan-table-body">
              <tr
                v-for="lecWeekPlan in lecWeekPlanList"
                :key="lecWeekPlan.lecWeekPlanId"
                class="lecture-plan-table-body-row"
              >
                <th class="lecture-plan-table-body-cell">
                  {{ `${lecWeekPlan.lecWeekRound}주차` }}
                </th>
                <th class="lecture-plan-table-body-cell">
                  {{ lecWeekPlan.lecWeekGoal }}
                </th>
                <th class="lecture-plan-table-body-cell">
                  {{ lecWeekPlan.lecWeekContent }}
                </th>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="btn-container-row">
          <button v-if="showApplicationBtn" id="applcation-btn" @click="lectureRegister">
            신청
          </button>
          <button id="close-btn" @click="closeModal">취소</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
@import './styled.css';
</style>
