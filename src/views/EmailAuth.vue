<template> <!-- 이메일 인증 화면 -->
  <v-container>
    <v-form>
      <v-row>
        <v-col cols="4" offset="4">
          <v-card-title style="justify-content: center">이메일 인증</v-card-title> <!-- 제목 -->
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="6" offset="3">
          <v-divider></v-divider>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="4" offset="4">
          <v-form ref="form" v-model="valid" lazy-validation>
            <!-- 이메일 입력 부분 -->
            <v-text-field 
              v-model="info.email" 
              label="이메일을 입력하세요"
              :rules="email_rule"
              required
              :disabled="sendCode"
            ></v-text-field>
          </v-form>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="2" offset="5">
          <v-btn @click="firstcheck()" style="width: 100%">코드 전송</v-btn> <!-- 코드 전송 버튼 -->
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="6" offset="3">
          <v-divider></v-divider>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="4" offset="4">
          <v-otp-input length="6" type="number" v-model="info.code"></v-otp-input> <!-- 이메일로 받은 코드 입력 부분 -->
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="2" offset="5">
          <v-btn @click="secondcheck()" style="width: 100%">인증 확인</v-btn> <!-- 인증 확인 버튼 -->
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="6" offset="3">
          <v-divider></v-divider>
        </v-col>
      </v-row>
    </v-form>

    <!-- 토스트 메시지 -->
    <v-snackbar v-model="snackbar" timeout="3000">
      {{ snackbarContents }}
      <template v-slot:action="{ attrs }">
        <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>

  </v-container>
</template>

<script>
import herokuAPI from '@/api/heroku.js';
import router from '@/router/index.js';

export default{
  data() {
    return {
    // 이메일 인증 정보
      info: {
        email: null,
        code: ""
      },

    // 인증 완료시 이동 화면
      next: null, 

    // 토스트 메시지 용
      snackbar: false,
      snackbarContents: "",

    // 이메일 전송 시 true, 이메일 입력 부분 비활성화용
      sendCode: false,

    // 유효성 검사
      valid: true,
      email_rule: [
        v => !!v || '인증코드를 받을 이메일을 입력하세요.',
        v => /^[a-zA-Z0-9+-_.]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$/.test(v) || '올바른 이메일을 입력하세요.',
      ]
    }
  },
  created() { // 뷰 생성시 동작
    console.log("route", this.$route.params.id); // params 받아오기, 0 : 회원가입, 1 : 비밀번호변경
    if(this.$route.params.id == 0) {
      this.next = "signup";
    } else if(this.$route.params.id == 1) {
      this.next = "changePassword";
    }
    console.log("params 메시지 받은내용", this.next);
  },
  methods: {
  // 입력 받은 이메일로 코드를 전송하는 부분
    firstcheck() { 
      let vm = this;
      const validate = this.$refs.form.validate();
      if(!validate) { // 형식 검사
        vm.snackbarContents = "잘못된 형식의 입력입니다."
        vm.snackbar = true;
        return;
      }
      const checkInfo = JSON.stringify({ // 전송할 정보 JSON 변환
        "email": this.info.email,
        "code": 0
      });
      console.log(checkInfo);
      herokuAPI.firstcheck(checkInfo) 
        .then(function (response) { // 서버로부터 전송 완료 응답
          console.log("firstcheck", response);
          vm.snackbarContents = "이메일을 전송했습니다."
          vm.snackbar = true;
          vm.sendCode = true;
        })
        .catch(function (e) { // 전송 실패 응답
          if(e.response.status == 501) {
            console.log("501 전송실패");
            vm.snackbarContents = "이메일 전송에 실패했습니다."
            vm.snackbar = true;
          } else if(e.response.status == 404) {
            console.log("404 존재하지 않는 이메일");
            vm.snackbarContents = "존재하지 않는 이메일입니다."
            vm.snackbar = true;
          }
        });
    },
  // 이메일로 받은 인증코드를 확인하는 부분
    secondcheck() {
      let vm = this;
      const codenum = Number(this.info.code); // 입력한 코드 숫자로 변환
      const checkInfo = JSON.stringify({ // JSON 변환
        "email": this.info.email,
        "code": codenum
      });
      const email = this.info.email;
      const next = this.next;
      console.log(checkInfo);
      herokuAPI.secondcheck(checkInfo)
        .then(function (response) {
          console.log("secondcheck", response);
          if(response.status == 200) { // 코드 일치시
            console.log("코드 일치", next); 
            if(next == 'signup') { // 회원가입인 경우
              localStorage.setItem("email", email); // 이메일 정보 로컬 스토리지에 저장 해 건네줌
            }
            router.push({name: next}); // 다음 화면으로 이동 (회원가입 또는 비밀번호 변경)
          }
        }) 
        .catch(function (e) { // 코드 불일치 또는 서버, 알 수 없는 오류
          if(e.response.status == 400) {
            console.log("400 잘못된 코드");
            vm.snackbarContents = "잘못된 인증코드입니다."
            vm.snackbar = true;
          } else if(e.response.status == 500) {
            console.log("500 등록 실패");
            vm.snackbarContents = "DB 오류로 등록에 실패했습니다."
            vm.snackbar = true;
          } else if(e.response.status == 502) {
            console.log("502 Unknown error");
            vm.snackbarContents = "알 수 없는 이유로 등록에 실패했습니다."
            vm.snackbar = true;
          }
        });
    }
  },
}
</script>