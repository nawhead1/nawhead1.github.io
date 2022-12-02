<template> <!-- 요리 사진 열람 화면 -->
  <v-container>
  
    <v-row justify="center">
      <v-col class="col-xl-8 col-md-10">
        <v-card height="md-800 xl-1000" color="#f5efe6">
          <!-- 요리 사진 게시판으로 돌아가기 -->
          <v-btn text to="/photo" class="ml-5 mt-5"> - 요리 사진 게시판</v-btn>
            <v-row>
              <v-col>
                <v-card fill-height color="#f5efe6" class="mt-10" flat>
                  <v-row>
                    <v-col cols="3" class="py-0 my-0">
                      <v-card fill-height color="#f5efe6" flat>
                        <v-card-title primary-title style="color:#7895B2">
                          작성자
                        </v-card-title>
                      </v-card>
                    </v-col>
                    <v-col class="py-0 my-0">
                      <v-card fill-height color="#f5efe6" flat>
                        <v-card-title primary-title >
                          {{ requestPhoto.nickname }} <!-- 작성자 닉네임 -->
                        </v-card-title>
                      </v-card>
                    </v-col>
                  </v-row>

                  <v-row>
                    <v-col cols="3" class="py-0 my-0">
                      <v-card fill-height color="#f5efe6" flat style="color:#7895B2">
                        <v-card-title>
                          작성일
                        </v-card-title>
                      </v-card>
                    </v-col>
                    <v-col class="py-0 my-0">
                      <v-card fill-height color="#f5efe6" flat>
                        <v-card-title primary-title>
                          {{ requestPhoto.upload_time.split(/[T]/)[0]}} <!-- 작성일 -->
                        </v-card-title>
                      </v-card>
                    </v-col>
                  </v-row>

                </v-card>
              </v-col>
            </v-row>
            
            <!-- 사진  -->
            <v-row>
              <v-col offset="2" cols="8">
                <v-img
                    max-height="350"
                    contain

                    :src="'data:image/jpeg;base64,'+requestPhoto.photo_link"
                  ></v-img>
              </v-col>
          
              <v-col cols="2" class="d-flex align-end">
                <v-row justify="start">
                  <div class=".buttons">
                    <v-btn v-if="isMine" @click="deletePopup" icon x-large>  <!-- 삭제 버튼 -->
                      <v-icon x-large>mdi-delete-outline</v-icon>
                      <div>삭제</div>
                    </v-btn>
                    
                    <v-card height="20" color="#f5efe6" flat></v-card>
                    <v-btn v-if="!isMine" @click="showMailCreate" icon x-large> <!-- 쪽지 버튼 -->
                      <v-icon x-large>mdi-email-arrow-right-outline</v-icon>
                      <div>쪽지</div>
                    </v-btn>
                    
                    <v-card height="20" color="#f5efe6" flat></v-card>
                    <v-btn v-if="!isMine" @click="showReportDialog" icon x-large> <!-- 신고 버튼 -->
                      <v-icon x-large>mdi-alert-octagon</v-icon>
                      <div>신고</div>
                    </v-btn>
                  </div>
                    
                </v-row>
                
              </v-col>
            </v-row>

            <v-row justify="center">
              <v-card color="#f5efe6" height="50" flat >
                <v-btn @click="likePhoto" icon x-large> <!-- 좋아요 버튼 -->
                  <v-icon x-large>mdi-thumb-up-outline</v-icon>
                  <div v-if="!isLikedAfter">좋아요</div> <!-- 좋아요를 하지 않은 상태인 경우 -->
                  <div v-if="isLikedAfter">좋아요 취소</div> <!-- 좋아요를 한 상태인 경우 -->
                </v-btn>
              </v-card>
            </v-row>

            <v-row justify="center" class="mt-10 thumbs">
              좋아요 수 {{ requestPhoto.like_count }} <!-- 좋아요 수 -->
            </v-row>
        </v-card>
      </v-col>
    </v-row>

    <!-- 팝업창 -->
    <v-dialog
      max-width="300"
      v-model="popupDialog"
    >
      <popup-dialog
        :headerTitle=headerTitle
        :btn1Title=btn1Title
        :btn2Title=btn2Title
        :btn2=btn2
        @hide="hideDialog"
        @submit="checkDialog"
      >
        <template v-slot:body>
          <div>{{ content1 }}<br>{{ content2 }}</div>
        </template>
      </popup-dialog>
    </v-dialog>

    <!-- 쪽지 작성 팝업창 -->
    <v-dialog
      max-width="500"
      v-model="createMailDialog"
    >
      <create-mail-dialog
        :receiverFixed="true"
        :receiver=requestPhoto.nickname
        @hide="hideMailCreate"
      />
    </v-dialog>

    <!-- 신고 팝업창 -->
    <v-dialog
      max-width="400"
      v-model="reportDialog"
    >
      <report-dialog
        :postType="2"
        :postID=requestPhoto.post_id
        @hide="hideReportDialog"
      />
    </v-dialog>

  </v-container>
</template>

<script>
import herokuAPI from '@/api/heroku.js';
import router from '@/router/index.js';
import PopupDialog from '@/components/popup.vue';
import ReportDialog from '@/components/report.vue';
import CreateMailDialog from '@/components/createMail.vue';

export default{
  components: {
    PopupDialog,
    ReportDialog,
    CreateMailDialog
  },
  data(){
    return{
    // 팝업창
      popupDialog: false,
      reportDialog: false,
      createMailDialog: false,
      headerTitle: "",
      content1: "",
      content2: "",
      btn1Title: "",
      btn2Title: "",
      btn2: false,

      requestPhoto: [],
      isLikedBefore: null,
      isLikedAfter: null,
      isMine: null,
      deletePost: false,
    }
  },
  mounted() { // 뷰 생성시 요리 사진 게시글 요청해서 받아오는 과정
    let pid = this.$route.params.id;
    console.log("post_id", pid);
    if(pid == null) {
      console.log("ERROR");
    }
    const UserInfo = JSON.parse(localStorage.getItem("UserInfo"));
    let vm = this;
    herokuAPI.photoLookup(pid, UserInfo.nickname)
      .then(function(response) {
        console.log("응답 온거", response);
        if(response.status == 200) {
            console.log("조회 성공");
            vm.requestPhoto = response.data.photoInfo;
            vm.isLikedBefore = response.data.likeInfo;
            vm.isLikedAfter = vm.isLikedBefore;
            if(vm.requestPhoto.nickname == UserInfo.nickname) {
              vm.isMine = true;
            } else {
              vm.isMine = false;
            }
          }
      })
      .catch(function (e) {
        if(e.response.status == 500) {
          console.log("500 DB 오류");
          vm.requestFailPopup();
        } else if(e.response.status == 502) {
          console.log("502 Unknown error");
          vm.requestFailPopup();
        }
      });
  },
  beforeDestroy() { // 뷰 없어지기 전 좋아요 상태 변화 확인, 등록 과정
    let vm = this;
    if (vm.deletePost) return;
    if (vm.isLikedBefore == vm.isLikedAfter) return; // 좋아요 상태 같은 경우
    const UserInfo = JSON.parse(localStorage.getItem("UserInfo"));
    let task = "";
      if(vm.isLikedBefore) task = "취소"; // 좋아요 취소
      else task = "등록"; // 좋아요 등록

      console.log(task);
      const likeInfo = JSON.stringify({
        "like_id": 0,
        "nickname": UserInfo.nickname,
        "postType": 2,
        "postId": vm.requestPhoto.post_id,
        "task": task
      });
      herokuAPI.photoLike(likeInfo)
        .then(function (response) {
          if(response.status == 200) console.log("좋아요 " + task + " 성공");
        })
        .catch(function (e) {
          if(e.response.status == 500) {
            console.log("500 취소 오류");
            vm.likeFailPopup(task);
          } else if(e.response.status == 501) {
            console.log("501 등록 오류");
            vm.likeFailPopup(task);
          } else if(e.response.status == 502) {
            console.log("502 Unknown error");
            vm.likeFailPopup(task);
          }
        });
  },
  methods: {
    showDialog() { // 팝업창 보이기
      this.popupDialog = true;
    },
    hideDialog() { // 팝업창 숨기기
      this.popupDialog = false;
    },
    deletePopup() { // 삭제 확인 팝업
      this.headerTitle = "요리 사진 게시글 삭제";
      this.content1 = "게시글을 삭제하시겠습니까?";
      this.btn1Title = "취소";
      this.btn2Title = "삭제";
      this.btn2 = true;
      this.showDialog();
    },
    deleteFailPopup() { // 삭제 실패 팝업
      this.headerTitle = "서버 오류";
      this.content1 = "게시글 삭제를 실패하였습니다.";
      this.btn1Title = "확인";
      this.btn2 = false;
      this.showDialog();
    },
    requestFailPopup() { // 게시글 요청 실패 팝업
      this.headerTitle = "요청 실패";
      this.content1 = "게시글을 불러오지 못했습니다.";
      this.content2 = "";
      this.btn1Title = "확인";
      this.btn2 = false;
      this.showDialog();
    },
    likeFailPopup(text) { // 좋아요 등록, 실패 팝업
      this.headerTitle = "서버 오류";
      this.content1 = '"좋아요" '+text+"에 실패하였습니다.";
      this.btn1Title = "확인";
      this.btn2 = false;
      this.showDialog();
    },
    checkDialog() { // 팝업창 버튼 클릭시
      this.deletePhoto();
      this.hideDialog();
    },
    deletePhoto() { // 요리사진 삭제 
      let vm = this;
      vm.deletePost = true;
      const deleteTarget = {
        "post_id": vm.requestPhoto.post_id,
        "photo_link": "",
        "like_count": 0,
        "upload_time": "",
        "nickname": vm.requestPhoto.nickname
      }
      herokuAPI.photoDelete(deleteTarget) 
        .then(function (response) {
          if(response.status == 200) {
            router.push({name: 'photo'});
          }
        })
        .catch(function (e) {
          if(e.response.status == 500) {
            console.log("500 DB 오류");
            vm.deleteFailPopup();
          } else if(e.response.status == 502) {
            console.log("502 Unknown error");
            vm.deleteFailPopup();
          }
        });
    },

    showReportDialog() { // 신고 팝업창 보이기
      this.reportDialog = true
    },
    hideReportDialog() { // 신고 팝업창 숨기기
      this.reportDialog = false
    },

    showMailCreate(){ // 쪽지 팝업창 보이기
      this.createMailDialog = true;
    },
    hideMailCreate(){ // 쪽지 팝업창 숨기기
      this.createMailDialog = false;
    },

    likePhoto() { // 좋아요 버튼 클릭시 동작, 서버랑 통신은 화면을 벗어날 때 초기와 상태가 다를 경우에만 실시
      this.isLikedAfter = !this.isLikedAfter;
      if(this.isLikedAfter) ++this.requestPhoto.like_count;
      else --this.requestPhoto.like_count;
    },
  }
}
</script>

<style>
.back{
  color: aqua;
}
.v-btn__content{
  display: flex;
  flex-direction: column;
}
.thumbs{
  color: rgb(248, 62, 62);
  font-size: 1.7em;
  font-weight: 600;
}
</style>