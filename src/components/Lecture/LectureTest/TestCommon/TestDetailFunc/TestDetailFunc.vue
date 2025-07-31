<script>
import { useUserInfo } from '@/stores/loginInfoState';
import axios from 'axios';

const { user } = useUserInfo();

const searchTestInfoDetail = (testId, lecId) => {
  const param = {
    testId: testId,
    lecId: lecId,
  };

  axios.post('/api/lecture/testQuestionInfoDetail.do', param).then((res) => {
    const lecInstructorId = res.data.lecInstructorId;
    const questionList = res.data.testQuestionInfoDetail;
    const optionList = res.data.testQuestionOptionDetail;
    const answerList = res.data.testQuestionAnswerInfoDetail;

    const isInstructor = user.userType === 'T' && user.loginId === lecInstructorId;

    const originalQuestionData = {
      questionList: questionList,
      optionList: optionList,
      answerList: answerList,
    };

    if (isInstructor) {
    }
  });
};

export default searchTestInfoDetail;
</script>
