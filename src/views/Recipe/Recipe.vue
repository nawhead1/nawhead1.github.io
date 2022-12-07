<template> <!-- 레시피 게시판 화면 -->
  <v-container>
    <v-row justify="center">
      <v-col class="col-xl-8 col-md-10">
        <!-- 상단 검색 부분 -->
        <v-card min-height="100" color="#f5efe6" class="mb-3">
          <!-- 위쪽 : 키워드 검색 부분 -->
          <div class="find-cook flex align-end mb-2">
            <!-- 검색 타입 선택 드롭다운 -->
            <dropdown class="my-dropdown-toggle my-0 ml-5"
              :options="search_standard"
              :selected="search_object" 
              v-on:updateOption="searchStandardRunOnSelect" 
              :placeholder="'검색 기준'"
              :closeOnOutsideClick="true">
            </dropdown>
            <!-- 검색어 입력 부분 -->
            <v-text-field
              label="검색어를 입력하세요"
              v-model="searchText"
              hide-details="auto"
              class="mx-5"
            ></v-text-field>
            <!-- 검색 버튼 -->
            <v-btn @click="beforeSelectKeyword" class="mx-3" color="#E8DFCA">
              검색
            </v-btn>
          </div>

          <div class="black-line mx-3"></div>

          <!-- 아래쪽 : 카테고리 검색 부분 -->
          <div class="category-search-dropdown mt-2 flex-column">
            <!-- 카테고리 선택 팝업창 버튼 -->
            <v-btn color="#f5efe6" depressed @click="showCategoryDialog">카테고리 검색</v-btn> 
          </div>
        </v-card>

        <v-card min-height="calc(100vh - 233px)" color="#f5efe6">
          <!-- 상부 레이아웃 -->
          <div class="recipe-top d-flex justify-space-between align-center pa-5">
            <!-- 레시피 게시판 타이틀 -->
            <span style="color:#7895B2; font-weight:900; font-size:1.3em;">레시피 게시판</span>
            <!-- 정렬 기준 선택 드롭다운 -->
            <dropdown class="my-dropdown-toggle"
              :options="sort_standard" 
              :selected="sort_object" 
              v-on:updateOption="sortStandardRunOnSelect" 
              :placeholder="'정렬 기준'"
              :closeOnOutsideClick="true">
            </dropdown>
          </div>

          <!-- 글쓰기 버튼 -->
          <v-btn fab to="/recipe/create" x-large color="primary" class="write-icon">
            <v-icon dark>mdi-pencil-outline</v-icon>
          </v-btn>

          <!-- 검색 결과 없는 경우 화면 -->
          <v-row v-if="!isSearchExist" justify="center">
            <v-col cols="12">
              <p style="text-align:center; font-size:1.2em;" class="mt-10">
                검색결과 레시피가 존재하지 않습니다.
              </p>
            </v-col>
          </v-row>

          <!-- 검색 결과 있는 경우 화면 -->
          <div v-if="isSearchExist">
            <!-- 레시피 게시글 -->
            <v-card height="100" class="mx-5 mb-5" v-for="item in recipes" :key="item.post_id" @click="toLookup(item.post_id)">
              <!-- 게시글 블럭 상단부 -->
              <div class="d-flex align-center">
                <!-- 제목 -->
                <span class="mx-10 py-3" style="font-size:1.1em; font-weight:600; color:#7895B2">{{item.title}}</span>
                <!-- 매운맛 단계 -->
                <v-icon color="red" v-if="item.degree_of_spicy>=1">mdi-chili-mild</v-icon>
                <v-icon color="red" v-if="item.degree_of_spicy>=2">mdi-chili-mild</v-icon>
                <v-icon color="red" v-if="item.degree_of_spicy>=3">mdi-chili-mild</v-icon>
                <v-icon color="red" v-if="item.degree_of_spicy>=4">mdi-chili-mild</v-icon>
                <v-icon color="red" v-if="item.degree_of_spicy>=5">mdi-chili-mild</v-icon>
              </div>

              <!-- 중앙 구분선 -->
              <div style="border: 0.5px solid #7895B2;" class="mx-5"></div>

              <!-- 게시글 블럭 하단부 -->
              <div class="d-flex align-center justify-space-between">
                <div style="color:#7895B2" class="ml-10 py-3">
                  <!-- 작성일 -->
                  <span class="mr-3">{{item.show_upload_time}}</span>
                  <!-- 작성자 닉네임 -->
                  <span>{{item.nickname}}</span>
                </div>
                <div class="mr-6">
                  <!-- 좋아요 아이콘 / 수 -->
                  <v-icon color="red">mdi-thumb-up-outline</v-icon> {{item.like_count}}
                  <!-- 댓글 아이콘 / 수 -->
                  <v-icon color="blue" class="ml-2">mdi-comment-processing-outline</v-icon> {{item.comment_count}}
                  <!-- 조회수 아이콘 / 수 -->
                  <v-icon color="green" class="ml-2">mdi-eye-outline</v-icon> {{item.views}}
                </div>
              </div>
            </v-card>
          </div>
          
          <!-- 페이지 이동 -->
          <v-pagination 
            v-model="page" 
            :length="pageLength"
            class="pb-10"
            @input="handlePage"  
          >
          </v-pagination>
        </v-card>

      </v-col>
    </v-row>

    <!-- 카테고리 팝업창 -->
    <v-dialog
      max-width="300"
      v-model="categoryDialog"
    >
      <category-dialog
        @category="beforeSelectCategory"
        @hide="hideCategoryDialog"
      />
    </v-dialog>

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
          <div>{{ content1 }} <br> {{content2}}</div>
        </template>
      </popup-dialog>
    </v-dialog>

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

<style scoped>
.find-cook{
  display: flex;
}
.black-line{
  border: 0.4px solid #7895B2;
}
.category-search-dropdown{
  display: flex;
  justify-content: center;
}
.write-icon{
  position: fixed;
  bottom: 10%;
  right: 5%;
  z-index: 9;
}
.my-text{
  color:#7895B2;
}
</style>

<script>
import herokuAPI from '@/api/heroku.js';
import router from '@/router/index.js';
import dropdown from 'vue-dropdowns';
import PopupDialog from '@/components/popup.vue';
import CategoryDialog from '@/components/Category.vue'

export default{
  components: {
    PopupDialog,
    CategoryDialog,
    'dropdown': dropdown
  },
  data () {
    return {
    // 팝업창
      popupDialog: false,
      headerTitle: "",
      content1: "",
      content2: "",
      btn1Title: "확인",
      btn2: false,
      categoryDialog: false,

    // 토스트 메시지
      snackbar: false,
      snackbarContents: "",

    // 레시피 정보들
      recipes: [],
      recipeID: null,
      total_page: null,

    // 드롭다운
      // 검색 기준
      search_standard: [
        {name: '요리 이름'},
        {name: '작성자'},
        {name: '재료'},
      ],
      search_object: {
        name: '요리 이름',
      },
      // 정렬 기준
      sort_standard: [
        { name: '최근 순'},
        { name: '조회수 순'},
        { name: '좋아요 순'}
      ],
      sort_object: {
        name: '최근 순',
      },

    //현재 검색 기준
      currentSearchStandard : "요리 이름",
      searchText: "",
      searchTextStorage: "",
      searchTypeStorage: "",
      category: "",

    // 현재 서버에 요청한 정보
      currentRequestType : "정렬",

    //페이지네이션 길이
      pageLength : 1,

    //현재 페이지
      page : 1,

    //검색 존재 여부
      isSearchExist : true,

    }
  },
  mounted() { // 뷰 생성시 동작
    let vm = this;
    herokuAPI.recipeList(1) // 1페이지 요청
      .then(function(response) { // 성공시
        console.log("리스트 응답 온거", response);
        if(response.status == 200) {
            console.log("조회 성공");
            vm.pageLength = response.data.total_page;
            for(let i = 0; response.data.recipeList[i] != null; i++) {
              // 레시피 작성일 정보 형태 변형
              var parsedTime = response.data.recipeList[i].upload_time.split(/[T]/);
              response.data.recipeList[i].show_upload_time = parsedTime[0];

              vm.recipes.push(response.data.recipeList[i]);
            }
          }
      })
      .catch(function (e) { // 실패시
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
    requestFailPopup() { // 실패
      this.headerTitle = "요청 실패";
      this.content1 = "레시피 게시글을 요청에 실패했습니다.";
      this.content2 = "";
      this.showDialog();
    },
    searchRequestFailPopup() { // 검색 실패
      this.headerTitle = "요청 실패";
      this.content1 = "검색 결과 요청에 실패하였습니다.";
      this.content2 = "";
      this.showDialog();
    },
    sortRequestFailPopup() { // 정렬 실패
      this.headerTitle = "정렬 실패";
      this.content1 = "정렬된 게시글 목록을";
      this.content2 = "가져오는데 실패했습니다.";
      this.showDialog();
    },

  // 카테고리 팝업창 관련
    showCategoryDialog() { // 카테고리 팝업창 보이기
      this.categoryDialog = true;
    },
    hideCategoryDialog() { // 카테고리 팝업창 숨기기
      this.categoryDialog = false;
    },
    beforeSelectCategory(name) { // 카테고리 선택시 동작
      let vm = this;
      vm.currentRequestType = "카테고리";
      vm.category = name;
      vm.selectCategory(name, 1);
    },
    selectCategory(name, page) { // 카테고리 서버 통신 부분
      let vm = this;
      const searchInfo = JSON.stringify({
        "searchType": "카테고리",
        "categories": name,
        "keywordType": null,
        "keyword": null,
        "page": page
      });
      vm.recipes = [];
      herokuAPI.recipeSearch(searchInfo)
        .then(function(response) {
          console.log("응답 온거", response);
          if(response.status == 200) {
            console.log("검색 성공");
            for(let i = 0; response.data.recipeList[i] != null; i++) {
              vm.recipes.push(response.data.recipeList[i]);
            }
            vm.pageLength = response.data.total_page;
            if(response.data.recipeList.length == 0){
              vm.isSearchExist = false;
            }
          }
        })
        .catch(function (e) {
          if(e.response.status == 500) {
            console.log("500 DB 오류");
            vm.searchRequestFailPopup();
          } else if(e.response.status == 502) {
            console.log("502 Unknown error");
            vm.searchRequestFailPopup();
          }
        });
    },
    beforeSelectKeyword() { // 키워드 검색 클릭시 동작
      if(this.searchText == "") {
        this.snackbarContents = "키워드를 입력해주세요."
        this.snackbar = true;
      }
      this.searchTextStorage = this.searchText;
      this.searchTypeStorage = this.search_object.name;
      this.selectKeyword(1);
    },
    selectKeyword(page) { // 키워드 검색 서버 통신 부분
      let vm = this;
      vm.currentRequestType = "타이핑";
      const searchInfo = JSON.stringify({
        "searchType": "타이핑",
        "categories": null,
        "keywordType": vm.searchTypeStorage,
        "keyword": vm.searchTextStorage,
        "page": page
      });
      vm.recipes = [];
      console.log(searchInfo);
      herokuAPI.recipeSearch(searchInfo)
        .then(function(response) {
          console.log("응답 온거", response);
          if(response.status == 200) {
              console.log("검색 성공");
              for(let i = 0; response.data.recipeList[i] != null; i++) {
                vm.recipes.push(response.data.recipeList[i]);
              }
              if(response.data.recipeList.length == 0){
                vm.isSearchExist = false;
              }
              vm.pageLength = response.data.total_page;
            }
        })
        .catch(function (e) {
          if(e.response.status == 500) {
            console.log("500 DB 오류");
            vm.searchRequestFailPopup();
          } else if(e.response.status == 502) {
            console.log("502 Unknown error");
            vm.searchRequestFailPopup();
          }
        });
    },

  // 클릭한 레시피 게시글 열람 페이지로 이동
    toLookup(recipeID) {
      router.push({
        path: "/recipe/lookup/"+recipeID,
      })
    },

    sortRecipeList(page, order_by) { // 정렬 서버 통신 부분
      let vm = this;
      vm.recipes = [];
      vm.currentRequestType = "정렬"
      
      const sortInfo = JSON.stringify({
        "arrangeBy": order_by,
        "page": page
      });
      herokuAPI.recipeSort(sortInfo)
        .then(function (response) {
          console.log("응답 온거", response);
          if(response.status == 200) {
            console.log("정렬 성공");
            for(let i = 0; response.data.recipeList[i] != null; i++) {
              vm.recipes.push(response.data.recipeList[i]);
            }
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

    // 검색 드롭다운 선택시 실행되는 함수
    searchStandardRunOnSelect(payload) {
      this.object = payload;
      if (this.object.name == "요리 이름") {
        console.log("요리 이름 선택");
        this.search_object.name = '요리 이름';
        this.currentSearchStandard = '요리 이름';
      }else if(this.object.name == "작성자") {
        console.log("작성자 선택");
        this.search_object.name = '작성자';
        this.currentSearchStandard = '작성자';
      }else if(this.object.name == "재료") {
        console.log("재료 선택");
        this.search_object.name = '재료';
        this.currentSearchStandard = '재료';
      }
    },

    // 정렬 드롭다운 선택시 실행되는 함수
    sortStandardRunOnSelect(payload) {
      this.object = payload;
      if (this.object.name == "최근 순") {
        console.log("최근 순 선택");
        this.sort_object.name = "최근 순";

        this.page = 1;
        console.log(this.page, this.sort_object.name);
        this.sortRecipeList(this.page, this.sort_object.name);

      } else if (this.object.name == "조회수 순") {
        console.log("조회수 순 선택");
        this.sort_object.name = "조회수 순";

        this.page = 1;
        console.log(this.page, this.sort_object.name);
        this.sortRecipeList(this.page, this.sort_object.name);

      } else if (this.object.name == "좋아요 순") {
        console.log("좋아요 순 선택");
        this.sort_object.name = "좋아요 순";

        this.page = 1;
        console.log(this.page, this.sort_object.name);
        this.sortRecipeList(this.page, this.sort_object.name);

      }
    },

    //페이지네이션 함수, 해당 정렬 또는 검색 기준을 유지하여 다음 페이지를 요청함
    handlePage(page){
      let vm = this;
      if(vm.currentRequestType == "정렬") vm.sortRecipeList(page, vm.sort_object.name);
      else if (vm.currentRequestType == "카테고리") vm.selectCategory(vm.category, page);
      else if (vm.currentRequestType == "타이핑") vm.selectKeyword(page);
    }
  }
}
</script>