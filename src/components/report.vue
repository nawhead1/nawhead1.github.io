<template> <!-- 신고 팝업창 -->
  <v-card>
    <!-- 제목 부분 -->
    <v-card-title class="justify-start">
      신고 사유
    </v-card-title>

    <v-card-text>
      <v-textarea
        label="신고 사유를 작성하세요"
        v-model="reportContents"
        required
        outlined
      ></v-textarea>
    </v-card-text>

    <!-- 버튼 부분 -->
    <v-card-actions class="justify-center mr-2 pb-4">
      <v-btn 
        color="#7895B2"
        small
        @click="$emit('hide')"
      >
        취소하기
      </v-btn>
      <v-btn
        color="#7895B2"
        small
        @click="reportPopup"
      >
        신고하기
      </v-btn>
    </v-card-actions>

    <!-- 팝업창 -->
    <v-dialog
      max-width="300"
      v-model="popupDialog"
    >
      <popup-dialog
        :headerTitle=titlePopup
        :btn1Title=titleBtn1
        :btn2Title=titleBtn2
        :btn2=btn2
        @hide="hideDialog"
        @submit="checkDialog"
      >
        <template v-slot:body>
          <div> {{ content }} </div>
        </template>
      </popup-dialog>
    </v-dialog>

  </v-card>
</template>

<script>
import PopupDialog from '@/components/popup.vue';
import herokuAPI from '@/api/heroku.js';

export default{
  name: "Report",
  components: {
    PopupDialog
  },
  data() {
    return {
      popupDialog: false,
      reportContents: "",
      titlePopup: "",
      content: "",
      titleBtn1: "",
      titleBtn2: "",
      btn2: true,
    };
  },
  props: {
    postType: {
      type: Number
    },
    postID: {
      type: Number
    },
  },
  methods: {
    showDialog(){ // 팝업창 보이기
      this.popupDialog = true
    },
    hideDialog(){ // 팝업창 숨기기
      this.popupDialog = false
    },
    checkDialog(){ // 팝업창 버튼 클릭시
      this.reportPost();
      this.hideDialog();
    },
    reportFailPopupR(){
      this.titlePopup = "등록 실패";
      this.content = "신고 등록에 실패했습니다.";
      this.titleBtn1 = "확인";
      this.btn2 = false;
      this.showDialog();
    },
    reportFailPopupP(){
      this.titlePopup = "서버 오류";
      this.content = "신고 정보 등록을 실패하였습니다.";
      this.titleBtn1 = "확인";
      this.btn2 = false;
      this.showDialog();
    },
    reportFailPopupComment(){
      this.titlePopup = "댓글 신고";
      this.content = "댓글 신고를 실패하였습니다.";
      this.titleBtn1 = "확인";
      this.btn2 = false;
      this.showDialog();
    },
    reportPopup(){
      this.titlePopup = "신고 확인";
      let reportType = "게시글";
      if(this.postType == -1) reportType = "댓글";
      this.content = reportType+"을 신고하시겠습니까?";
      this.titleBtn1 = "취소";
      this.titleBtn2 = "신고";
      this.btn2 = true;
      this.showDialog();
    },
    emitMethod(){
      this.$emit('hide');
    },
    reportPost(){ // 신고 등록 로직
        let vm = this;
        console.log("통합 신고 로직"); // 레시피->1, 댓글->-1, 요리사진->2
        const UserInfo = JSON.parse(localStorage.getItem("UserInfo"));
        const reportInfo = JSON.stringify ({
          "id": 0,
          "contents": vm.reportContents,
          "post_type": vm.postType,
          "post_id": vm.postID,
          "reporter": UserInfo.nickname
        });
        console.log("신고때 보낼 거", reportInfo);
        if(vm.postType == 1) {
          herokuAPI.recipeReport(reportInfo)
          .then(function (response) {
            if(response.status == 200) {
              console.log("레시피 게시글 신고 성공");
              vm.emitMethod();
            }
          })
          .catch(function (e) {
            if(e.response.status == 500) {
              console.log("500 DB error");
              vm.reportFailPopupR();
            } else if(e.response.status == 502) {
              console.log("502 Unknown error");
              vm.reportFailPopupR();
            }
          });
        } else if(vm.postType == 2) {
          herokuAPI.photoReport(reportInfo)
          .then(function (response) {
            if(response.status == 200) {
              console.log("요리사진 게시글 신고 성공");
              vm.emitMethod();
            }
          })
          .catch(function (e) {
            if(e.response.status == 500) {
              console.log("500 DB error");
              vm.reportFailPopupP();
            } else if(e.response.status == 502) {
              console.log("502 Unknown error");
              vm.reportFailPopupP();
            }
          });
        } else if(vm.postType == -1) {
          herokuAPI.commentReport(reportInfo)
          .then(function (response) {
            console.log("응답 정보", response);
            if(response.status == 200) {
              console.log("댓글 신고 성공");
              vm.emitMethod();
            }
          })
          .catch(function (e) {
            if(e.response.status == 404) {
              console.log("404 DB error");
              vm.reportFailPopupComment();
            } else if(e.response.status == 500) {
              console.log("500 Unknown error");
              vm.reportFailPopupComment();
            }
          });
        }
    },
  }
};
</script>