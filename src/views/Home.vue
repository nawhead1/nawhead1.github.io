<template> <!-- 홈 화면 -->
  <v-container>
    <v-row justify="center">
      <v-col class="col-xl-8 col-md-10">
        <v-card min-height="400" color="#f5efe6" class="pa-3">
          <!-- 상부 추천 레시피 부분 제목 -->
          <v-card-title class="my-text recommend-recipe-title pa-5">
            추천 레시피
          </v-card-title>

          <!-- 추천 레시피 -->
          <v-card height="100" class="mx-5 mb-5" v-for="item in recipes" :key="item.title" color="#E8DFCA" @click="toLookup(item.post_id)">
            <div class="d-flex align-center">
              <span class="mx-10 py-3" style="font-size:1.1em; font-weight:600; color:#7895B2">{{item.title}}</span> <!-- 제목 -->
              <v-icon color="red" v-if="item.degree_of_spicy>=1">mdi-chili-mild</v-icon> <!-- 매운맛 고추 아이콘 표시 -->
              <v-icon color="red" v-if="item.degree_of_spicy>=2">mdi-chili-mild</v-icon>
              <v-icon color="red" v-if="item.degree_of_spicy>=3">mdi-chili-mild</v-icon>
              <v-icon color="red" v-if="item.degree_of_spicy>=4">mdi-chili-mild</v-icon>
              <v-icon color="red" v-if="item.degree_of_spicy>=5">mdi-chili-mild</v-icon>
            </div>
            <div style="border: 0.5px solid #7895B2;" class="mx-5"></div>
            <div class="d-flex align-center justify-space-between">
              <div style="color:#7895B2" class="ml-10 py-3">
                <span class="mr-3">{{item.upload_time}}</span> <!-- 작성 일자 -->
                <span>{{item.nickname}}</span> <!-- 게시글 작성자 닉네임 -->
              </div>
              <div class="mr-6">
                <v-icon color="red">mdi-thumb-up-outline</v-icon> {{item.like_count}} <!-- 좋아요 아이콘 / 수 -->
                <v-icon color="blue" class="ml-2">mdi-comment-processing-outline</v-icon> {{item.comment_count}} <!-- 댓글 아이콘 / 수 -->
                <v-icon color="green" class="ml-2">mdi-eye-outline</v-icon> {{item.views}} <!-- 조회수 아이콘 / 수 -->
              </div>
            </div>
          </v-card>

        </v-card>

        <v-card min-height="200" color="#f5efe6" class="pa-3 mt-3 overflow-auto">
          <!-- 하부 추천 사진 부분 제목 -->
          <v-card-title class="my-text recommend-recipe-title pa-5">
            추천 요리 사진
          </v-card-title>
          
          <!-- 추천 요리 사진 -->
          <div class="d-flex align-center photo-box">
            <div v-for="item in photo" :key="item.photo_link" @click="toLookupPhoto(item.post_id)">
              <v-img
                contain
                max-width="300"
                :src="'data:image/jpeg;base64,'+item.photo_link"
                class="mx-4"
              ></v-img> <!-- 요리 사진 -->
            </div>
          </div>
          
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped>
.my-text{
  color: #42688e;
}
.recommend-recipe-title{
  color: #7ca5c9;
  font-weight: 900;
}
.photo-box{
  cursor: pointer;
}
</style>

<script>
import herokuAPI from '@/api/heroku.js';
import router from '@/router/index.js';

export default{
  data() {
    return {
    // 요리사진 정보
      photo: [],
      photoID: null,

    // 레시피 정보
      recipes: [],
      recipeID: null,
      total_page: null,

    // 페이지네이션 길이
      pageLength : 1,
    };
  },
  mounted() { // 뷰 생성시 동작
  let vm = this;
    herokuAPI.recipeList(1) // 레시피 5개 받아오는 동작
      .then(function(response) {
        console.log("리스트 응답 온거", response);
        if(response.status == 200) {
            console.log("조회 성공");
            vm.pageLength = response.data.total_page;
            for(let i = 0; i < 5; i++) {
              var parsedTime = response.data.recipeList[i].upload_time.split(/[T]/);
              response.data.recipeList[i].upload_time = parsedTime[0];
              vm.recipes.push(response.data.recipeList[i]);
            }
          }
      })
      .catch(function (e) {
        if(e.response.status == 500) {
          console.log("500 DB 오류");
        } else if(e.response.status == 502) {
          console.log("502 Unknown error");
        }
      });

      herokuAPI.photoList(1) // 요리 사진 5개 받아오는 동작
      .then(function(response) {
        console.log("응답 온거", response);
        if(response.status == 200) {
          console.log("조회 성공");
          vm.pageLength = response.data.total_page;
          console.log(vm.pageLength);
          for(let a = 0; a < 5; a++) {
            vm.photo.push(response.data.photoList[a]);
          }
          console.log(vm.photo);
        }
      })
      .catch(function (e) {
        if(e.response.status == 500) {
          console.log("500 DB 오류");
        } else if(e.response.status == 502) {
          console.log("502 Unknown error");
        }
      });
  },
  methods: {
    toLookup(recipeID) { // 레시피 클릭 시 동작
      router.push({
        path: "/recipe/lookup/"+recipeID, // 해당 레시피 열람 화면으로 이동
      })
    },
    toLookupPhoto(photoID) { // 요리사진 클릭 시 동작
      router.push({
        path: "/photo/lookup/"+photoID, // 해당 요리사진 열람 화면으로 이동
      })
    },
  }
}

</script>