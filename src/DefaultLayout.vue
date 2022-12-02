<template> <!-- 헤더 있는 레이아웃 -->
  <v-card class="overflow-hidden">
    <v-app-bar
      color="#f5efe6"
      app dense fixed
    >

      <!-- 헤더 윗부분 -->
      <v-col offset="2" cols="8">
        <div style="width:100%" class="d-flex justify-space-between align-center"> <!-- 레쉽피 로고, 클릭 시 홈화면으로 이동 -->
          <v-img 
            contain
            max-width="40"
            src="@/fav.png"
            class="ml-9"
          >
            <v-btn plain retain-focus-on-click v-ripple="false" to="/"></v-btn>
          </v-img>
          <v-btn text @click="showDialog" class="mr-6 my-text">로그아웃</v-btn> <!-- 로그아웃 버튼 -->
        </div>
      </v-col>
      

      <v-spacer />

      <template v-slot:extension>
        <v-row>
          <v-spacer />
          <v-divider vertical inset style="height:20px"></v-divider>
          <v-col> <!-- 레시피 게시판 이동 버튼 -->
            <v-btn to="/recipe" text style="width: 100%" class="my-text my-vertical-line">레시피 게시판</v-btn>
          </v-col>
          <v-divider vertical inset style="height:20px"></v-divider>
          <v-col> <!-- 요리 사진 게시판 이동 버튼 -->
            <v-btn to="/photo" text style="width: 100%" class="my-text my-vertical-line">요리 사진 게시판</v-btn>
          </v-col>
          <v-divider vertical inset style="height:20px"></v-divider>
          <v-col> <!-- 마이페이지 이동 버튼 -->
            <v-btn to="/mypage" text style="width: 100%" class="my-text my-vertical-line">마이페이지</v-btn>
          </v-col>
          <v-divider vertical inset style="height:20px"></v-divider>
          <v-spacer />
        </v-row>
      </template>
    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>

    <!-- 팝업창 -->
    <v-dialog
      max-width="300"
      v-model="popupDialog"
    >
      <popup-dialog
        headerTitle="로그아웃"
        btnTitle="취소"
        btn2Title="확인"
        @hide="hideDialog"
        @submit="checkDialog"
      >
        <template v-slot:body>
          <div>정말 로그아웃 하시겠습니까?</div>
        </template>
      </popup-dialog>
    </v-dialog>

  </v-card>
</template>

<style scoped>
.my-text{
  color: #7895B2;
}
.my-vertical-line{
  margin-top: -15px;
  color: #7895B2;
}
</style>

<script>
import router from '@/router/index.js';
import PopupDialog from '@/components/popup.vue';

export default{
  components: {
    PopupDialog
  },
  data(){
    return{
      popupDialog: false,
    }
  },
  methods: {
    showDialog(){ // 팝업창 보이기
      this.popupDialog = true
    },
    hideDialog(){ // 팝업창 숨기기
      this.popupDialog = false
    },
    checkDialog(){ // 팝업창 버튼 클릭시 동작
      this.logout();
      this.hideDialog();
    },
    logout() { // 로그아웃 기능
      localStorage.removeItem("UserInfo"); // 로컬스토리지에 저장해 둔 유저 정보 삭제
      router.push({name: 'login'}); // 로그인 화면으로 이동
    }
  }
}
</script>