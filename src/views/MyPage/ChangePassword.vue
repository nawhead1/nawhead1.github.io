<template> <!-- 비밀번호 변경 화면 -->
  <v-container>
    <v-layout>
      <v-row justify="center">
        <v-col cols="8">
          <v-card color="#f5efe6" height="800">
            <v-card color="#f5efe6" height="200" flat>
              
            </v-card>
            <v-layout>
              <v-row justify="center">
                <v-col class="col-xl-4 col-md-6">
                  <v-card color="#fefefe" height="400" rounded="xl">
                    <v-card-title id="title" class="pa-10 justify-center fontsize">비밀번호 변경</v-card-title>

                    <v-col>
                      <v-form ref="form" v-model="valid" lazy-validation>
                        <v-text-field 
                          v-model="user.pw" 
                          filled 
                          label="새 비밀번호" 
                          :rules="pw_rule"
                          type="password"
                          placeholder="새로운 비밀번호 입력."
                          required
                        ></v-text-field>
                        <v-text-field 
                          v-model="user.pwcheck" 
                          filled 
                          label="비밀번호 확인" 
                          :rules="pwch_rule"
                          type="password"
                          placeholder="비밀번호 확인."
                          required
                        ></v-text-field>
                      </v-form>
                    </v-col>
                    
                    <v-col>
                      <v-row justify="center">
                        <v-card-actions>
                          <v-btn outlined width="100" to="/mypage">취소</v-btn>
                        </v-card-actions>
                        <v-card-actions>
                          <v-btn outlined width="100" @click="PWchange()">변경</v-btn>
                        </v-card-actions>
                      </v-row>
                      
                    </v-col>
                  </v-card>
                </v-col>
              </v-row>
            </v-layout>
            
          </v-card>
        </v-col>
      </v-row>
    </v-layout>

    <!-- 팝업창 -->
    <v-dialog
      max-width="300"
      v-model="popupDialog"
    >
      <popup-dialog
        :headerTitle=headerTitle
        btn1Title="확인"
        :btn2=false
        @hide="hideDialog"
      >
        <template v-slot:body>
          <div> {{ content1 }} </div>
        </template>
      </popup-dialog>
    </v-dialog>

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

<style>
  #title{
    font-weight: 700;
    font-size: 2em;
  }

</style>

<script>
import herokuAPI from '@/api/heroku.js';
import router from '@/router/index.js';
import PopupDialog from '@/components/popup.vue';

export default{
  components: {
    PopupDialog
  },
  data() {
    return {
      valid: true,
    // 팝업창
      popupDialog: false,
      headerTitle: "",
      content1: "",

      snackbar: false,
      snackbarContents: "",

    // 사용자정보
      user: {
        email: '',
        id: '',
        nickname: '',
        pw: '',
        pwcheck: '',
        al: ''
      },
      nickname: '',

    // 유효성 검사
      pw_rule: [
        v => !!v || '새로운 비밀번호를 입력하세요.',
        v => !(v && v.length < 6) || '비밀번호는 6자 이상이여야 합니다.',
        v => /^[a-z0-9!?@<>]*$/.test(v) || '비밀번호는 영어 소문자, 숫자, 특수문자(!, ?, @, <, >)만 사용 가능합니다.',
        v => /^.*[a-z]+.*$/.test(v) || '비밀번호에는 영어 소문자가 포함되어야 합니다.',
        v => /^.*[0-9]+.*$/.test(v) || '비밀번호에는 숫자가 포함되어야 합니다.',
        v => /^.*[!?@<>]+.*$/.test(v) || '비밀번호에는 특수문자(!, ?, @, <, >)가 포함되어야 합니다.',
      ],
      pwch_rule: [
        v => !!v || '비밀번호 확인을 입력하세요.',
        v => v === this.user.pw || '비밀번호가 일치하지 않습니다.',
      ],
    };
  },
  created() {
    const UserInfo = JSON.parse(localStorage.getItem("UserInfo"));
    this.nickname = UserInfo.nickname
    this.user.id = UserInfo.uid
    this.user.email = UserInfo.email
    this.user.al = UserInfo.auto_login
  },
  methods: {
    showDialog(){ // 팝업창 보이기
      this.popupDialog = true
    },
    hideDialog(){ // 팝업창 숨기기
      this.popupDialog = false
    },
    PWchange() {
      const vm = this;
      const validate = this.$refs.form.validate();
      if(!validate) {
        vm.headerTitle = "양식 오류";
        vm.content1 = "양식에 맞지않은 비밀번호입니다.";
        vm.showDialog();
        return;
      }
      let hashedPWD = this.hashing(vm.user.pw);

      const userInfo = JSON.stringify({
        "nickname": this.nickname,
        "uid": this.user.id,
        "password": hashedPWD,
        "email": this.user.email,
        "auto_login": this.user.al,
      });
      console.log(userInfo);
      herokuAPI.changePW(userInfo)
        .then(function (response) {
          console.log("pwChange", response)
          if(response.status == 200) {
            console.log("비번변경 성공")
            localStorage.setItem("UserInfo", userInfo);
            router.push({name: 'mypage'});
          }
        }) 
        .catch(function (e) {
          if(e.response.status == 500) {
            console.log("500 비밀번호 변경 실패");
            vm.snackbarContents = "비밀번호 변경 요청 실패했습니다."
            vm.snackbar = true;
          } else if(e.response.status == 502) {
            console.log("502 Unknown error");
            vm.snackbarContents = "알 수 없는 오류로 비밀번호 변경 요청 실패했습니다."
            vm.snackbar = true;
          }
        });
      
    },
    hashing(content){
      var result = "";
      for(let i = 0; i < content.length; i++){
        result += String.fromCharCode(Number(content.charCodeAt(i))+1);
      }
      return result;
    },
    rhashing(content){
      var result = "";
      for(let i = 0; i < content.length; i++){
        result += String.fromCharCode(Number(content.charCodeAt(i))-1);
      }
      return result;
    }
  }
}
</script>