<template>
  <v-app>
    <router-view />
  </v-app>
</template>

<script>
import router from '@/router/index.js';

export default {
  name: 'App',

  data: () => ({
    
  }),
  created() { // 자동 로그인용 로직
    const UserInfo = JSON.parse(localStorage.getItem("UserInfo")); // 로컬 스토리지의 유저 정보 가져오기
    if(UserInfo == null) { // 유저 정보가 없는 경우
      router.push({name: 'login'}); // 로그인 한 정보 없는 경우, 로그인 화면으로 이동
    } else if(sessionStorage.getItem("running") != "true") { // 실행 중 새로고침 구분을 위해 세션 스토리지 정보 유무 확인
      if(UserInfo.auto_login == false) { // 유저 정보는 있는데 오토 로그인을 선택하지 않은 경우
        localStorage.removeItem("UserInfo"); // 로컬스토리지 기존 유저 정보 삭제
        router.push({name: 'login'}); // 로그인 화면으로 이동
      }
    }
  }
};
</script>
