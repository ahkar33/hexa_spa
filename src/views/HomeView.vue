<template>

  <section id="public-home-main" class="container-fluid p-0 row m-0">

    <Navbar @update:infos="updateInfo" :userInfo="userInfo" :isLogin="isLogin"/>

    <div v-if="categories != null" id="categories" class="container my-5 d-flex justify-content-center gap-2 flex-wrap">
      <Category v-for="category in categories" :key="category" :text="category.news_category_name" :id="category.news_category_id" />
    </div>

    <div v-if="todayNews.length" id="today-news" class="col-xl-9 col-md-10 col-sm-12 mx-auto text-start">
      <span class="h2 text-start text-danger fw-bold my-5">Today News</span>

      <div id="today-news-wrapper" class="container-fluid row my-4 gap-2 d-flex justify-content-center flex-wrap">
        <TodayNews v-for="news in todayNews" :key="news.news_id" :info="news"/>
      </div>

    </div>

    <!-- news -->
    <div v-if="allNews != null && paginate.length > 0" id="news-collection" class="col-xl-9 col-md-10 col-sm-12 mx-auto  text-start">
      <span class="h2 text-start text-danger fw-bold my-5">News</span>

      <div id="news-wrapper" class="container-fluid row my-4 gap-2 d-flex justify-content-center flex-wrap mx-auto">
        <div v-for="data in paginate" :key="data.news_id" class="news-box my-1 col-xl-3 col-lg-4 col-sm-10 text-light fw-bold p-0" id="news">
            <News :info="data"/>
        </div>
      </div>

      <Pagination @go:to="goTo" @go:prev="goPrev" @go:next="goNext" :curPage="curPage" :totalPage="totalPage" />
    </div>


    <div v-else class="p-5 text-center">
      Loading...
    </div>

    <!-- footer -->
    <HomeFooter/>
  </section>

</template>

<script>
import Navbar from '../components/public/Navbar.vue';
import Category from '../components/public/Category.vue';
import HomeFooter from '../components/public/HomeFooter.vue';
import TodayNews from '../components/public/TodayNews.vue';
import News from '../components/public/News.vue';
import Pagination from '../components/public/Pagination.vue';
import { mapActions , mapGetters } from 'vuex';
import getUpdateInfo from '@/composables/getUpdateInfo';
import getSetTitle from '@/composables/getSetTitle';

export default {
  name : 'HomeView',
  components : {
    Navbar,
    Category,
    HomeFooter,
    TodayNews,
    News,
    Pagination
  },
  data(){
    return {
      max : 11,
      curPage : 1,
    }
  },
  setup(){
    const { updateInfo } = getUpdateInfo();

    return { updateInfo  };
  },
  methods : {
    goTo(page){
      this.curPage = page ;
      localStorage.setItem('curPage',page);

    }, 
    goPrev(){
      this.curPage = this.curPage == 1 ? this.totalPage : this.curPage - 1; 
       localStorage.setItem('curPage',this.curPage);
    },
    goNext(){
       this.curPage = this.curPage == this.totalPage ? 1 : this.curPage + 1;
       localStorage.setItem('curPage',this.curPage);
    },
    ...mapActions(['getCategories','getTodayNews','getNewsInPage','getAllNews','getUserInfo']),
  },
  computed : {
    paginate(){
      const allNews = this.allNews;
      let start = (this.curPage - 1) * this.max;
      let end = start + this.max;
      let results = [];
    
      if(allNews != null ){
          for(let i = start ; i < end ; i++ ){
            if(allNews[i] != undefined){
              results.push(allNews[i]);
            }
        }
      }
      return results;
    },
    ...mapGetters(['categories','todayNews','allNews','totalPage','userInfo','isLogin']),
  },
  created(){
    if(localStorage.getItem('curPage') == null){
      this.curPage = 1;
      localStorage.setItem('curPage',this.curPage);
    }else{
      this.curPage = Number(localStorage.getItem('curPage'));
    }
  },
  mounted(){
    getSetTitle("hexa | home");

     this.getCategories();
     this.getTodayNews();
     this.getAllNews({ maxPerPage : this.max});
     this.getUserInfo();

     window.scrollTo(0,0);
  },
}
</script>