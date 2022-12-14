<template> <!-- 요리 사진 게시판 화면 -->
  <v-container>
    <v-row justify="center">
      <v-col class="col-xl-8 col-md-10">
        <v-card min-height="calc(100vh - 120px)" color="#f5efe6">

          <div class="recipe-top d-flex justify-space-between align-center pa-5">
            <!-- 제목 -->
            <span style="color:#7895B2; font-weight:900; font-size:1.3em;">요리 사진 게시판</span>
            <!-- 정렬 선택 -->
            <dropdown class="my-dropdown-toggle my-5 "
              :options="search_standard" 
              :selected= currentSearchOption
              v-on:updateOption="methodToRunOnSelect" 
              :placeholder="'정렬 기준'"
              :closeOnOutsideClick="true"
            ></dropdown>
          </div>

          <!-- 글쓰기 버튼 -->
          <v-btn fab to="/photo/create" x-large color="primary" class="write-icon">
            <v-icon dark>mdi-pencil-outline</v-icon>
          </v-btn>

          <v-row justify="center">
            <v-col cols="8">
              <!-- 요리 사진 게시글 -->
              <v-card height="400" v-for="item in photo" :key="item.post_id" class="my-10" @click="toLookup(item.post_id)">
                <v-row>
                  <v-col class="d-flex justify-space-between">
                    <div class="pl-5">
                      <v-card-title>{{item.nickname}}</v-card-title> <!-- 작성자 닉네임 -->
                      <v-card-subtitle>{{item.upload_time.split(/[T]/)[0]}}</v-card-subtitle> <!-- 작성 시간 -->
                    </div>
                    <div>
                      <v-icon class="pa-6" large> <!-- 좋아요 아이콘 -->
                        mdi-thumb-up-outline
                      </v-icon>
                      <span class="pr-6 like-count">{{item.like_count}}</span> <!-- 좋아요 수 -->
                    </div>
                  </v-col>
                </v-row>

                <div class="mx-6 mt-3">
                  <!-- 사진 -->
                  <v-img
                    max-height="250"
                    contain
                    :src="'data:image/jpeg;base64,'+item.photo_link"
                  ></v-img>
                </div>
                    
              </v-card>

              <!-- 페이지 이동 -->
              <v-pagination 
                v-model="page"
                :length="pageLength"
                class="pb-10"
                @input="handlePage"
              >
              </v-pagination>
            </v-col>
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
        :btn2=btn2
        @hide="hideDialog"
      >
        <template v-slot:body>
          <div>{{ content1 }}</div>
        </template>
      </popup-dialog>
    </v-dialog>

  </v-container>
</template>

<style>
.sort-base{
  font-size: 1.2em;
}
.like-count{
  font-size: 1.6em;
}
.write-icon{
  position: fixed;
  bottom: 10%;
  right: 5%;
  z-index: 9;
}
</style>

<script>
import herokuAPI from '@/api/heroku.js';
import router from '@/router/index.js';
import dropdown from 'vue-dropdowns';
import PopupDialog from '@/components/popup.vue';

export default{
  components: {
    'dropdown': dropdown,
    PopupDialog,
  },
  data() {
    return {
    //팝업창
      popupDialog: false,
      headerTitle: "",
      content1: "",
      btn1Title: "확인",
      btn2: false,

    //요리사진 정보들
      photo: [],
      photoID: null,

    // 정렬 드롭다운
      search_standard: [
          {name: '최근 순'},
          {name: '좋아요 순'}
      ],
      currentSearchOption : {
        name: "최근 순"
      },
      search_object: {
        name: "정렬 기준",
      },
    // 페이지
      page: 1,
    // 페이지 수
      pageLength: null,
    };
  },
  mounted() {
    let vm = this;
    herokuAPI.photoList(1)
      .then(function(response) {
        console.log("응답 온거", response);
        if(response.status == 200) {
          console.log("조회 성공");
          vm.pageLength = response.data.total_page;
          console.log(vm.pageLength);
          for(let i = 0; response.data.photoList[i] != null; i++) {
            vm.photo.push(response.data.photoList[i]);
          }
          console.log(vm.photo);
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
  methods: {
  // 팝업창 관련
    showDialog() { // 팝업창 보이기
      this.popupDialog = true;
    },
    hideDialog() { // 팝업창 숨기기
      this.popupDialog = false;
    },
    requestFailPopup() { // 요청 실패
      this.headerTitle = "요청 실패";
      this.content1 = "사진 게시글 요청을 실패하였습니다.";
      this.showDialog();
    },
    sortRequestFailPopup() { // 정렬 요청 실패
      this.headerTitle = "서버 오류";
      this.content1 = "게시글 요청을 실패하였습니다.";
      this.showDialog();
    },

  // 클릭한 요리사진 게시글 열람 페이지로 이동
    toLookup(photoID) {
      router.push({
        path: "/photo/lookup/"+photoID,
      })
    },

  //사진 리스트를 정렬해서 즉시 변경하는 함수
    sortPhotoList(page, order_by) {
      let vm = this;
      vm.photo = [];

      const sortInfo = JSON.stringify({
        "arrangeBy": order_by,
        "page": page
      });

      herokuAPI.photoSort(sortInfo)
        .then(function (response) {
          console.log("응답 온거", response);
          if(response.status == 200) {
            console.log("정렬 성공");
            for(let i = 0; response.data.photoList[i] != null; i++) {
              vm.photo.push(response.data.photoList[i]);
            }
            console("리스트 내용 : ", vm.photo);
            window.scrollTo({top:0});
          }
        })
        .catch(function (e) {
          if(e.response.status == 500) {
            console.log("500 DB 오류");
            vm.sortRequestFailPopup();
          } else if(e.response.status == 502) {
            console.log("502 Unknown error");
            vm.sortRequestFailPopup();
          }
        });
    },

  //정렬 드롭다운 선택시 실행되는 함수
    methodToRunOnSelect(payload) {
      this.object = payload;
      if (this.object.name == "최근 순") {
        console.log("최근 순 선택");
        this.currentSearchOption.name = "최근 순";
        this.page=1;

        this.sortPhotoList(this.page, this.currentSearchOption.name);
      } else if(this.object.name == "좋아요 순") {
        console.log("좋아요 순 선택");
        this.currentSearchOption.name = "좋아요 순";
        this.page=1;
        this.sortPhotoList(this.page, this.currentSearchOption.name);
      }
    },

  // 페이지네이션
    handlePage(){
      let vm = this;

      if(this.currentSearchOption.name == "최근 순") {
        this.sortPhotoList(vm.page, this.currentSearchOption.name);
      } else if(this.currentSearchOption.name == "좋아요 순") {
        this.sortPhotoList(vm.page, this.currentSearchOption.name);
      }
    },
  }
}
</script>