<template>
  <main-box :title="title" main-class="pad-20">
    <product-top-nav ctg="AIRPORT_BUS" :active="1" :nav1router="jumpToAirportPickSearch"></product-top-nav>
    <search-box
            :start-place="searchKeys.startPlace"
            :end-place="searchKeys.endPlace"
            @changeStation="jumpToAirportPickSearch"
            @jumpToFrom="jumpToCitySearch('startPlace')"
            @jumpToTo="jumpToAirportList">
      <group class="datetime-x line line-x-t">
        <calendar
                title="用车时间"
                :replace-text-list="{'TODAY':'今'}"
                :weeks-list="calendar.weeksList"
                :disable-past="calendar.disablePast"
                :display-format="formatDate"
                v-model="searchKeys.startTime">
        </calendar>
      </group>
    </search-box>
    <input type="button" class="button button-primary" @click="search" value="查询"/>
  </main-box>
</template>
<script>
  import MainBox from 'components/mainBox/index.vue'
  import ProductTopNav from 'appComponents/productTopNav/index.vue'
  import SearchBox from 'appComponents/searchBox/index.vue'
  import {Group, Calendar} from 'vux'
  import { mapState } from 'vuex'

  export default {
    components: {
      MainBox,
      ProductTopNav,
      SearchBox,
      Group,
      Calendar
    },
    data () {
      return {
        title: this.$route.meta.title,
        pageRouter: this.$route.name,
        pageId:'',
        calendar:{
          weeksList: ['日', '一', '二', '三', '四', '五', '六 '],
          disablePast:true
        },
        searchKeys: {
          startPlace: TOOL.placeHolder.startPlace,
          endPlace: TOOL.placeHolder.offAirport,
          airportCode:'',
          startTime: 'TODAY'
        }
      }
    },
    computed: {
      ...mapState({
        appCity: state => state.city.appCity,
      })
    },
    watch:{
      /**
       * 定位城市初始化赋值
       */
      appCity (newVal, val) {
        this.searchKeys.startPlace = newVal
      },
      $route () {
        this.initialize()
      }
    },
    mounted(){
      this.initialize()
    },
    methods:{
      initialize () {
        if(this.$route.query.pageName){
          this.title = this.$route.query.pageName
        }
        this.pageId = +this.$route.query.pageId || 3
  
        if(this.$route.query.airportCode){
          this.searchKeys.airportCode = this.$route.query.airportCode
        }
  
        if(this.$route.query.startPlace){
          this.searchKeys.startPlace = this.$route.query.startPlace
        }else{
          this.searchKeys.startPlace = this.appCity
        }
  
        if(this.$route.query.endPlace){
          this.searchKeys.endPlace = this.$route.query.endPlace
        }
  
        if(appStorage.get(this.pageRouter)){
          this.searchKeys = Object.assign(this.searchKeys, JSON.parse(appStorage.get(this.pageRouter)))
        }
        /**
         * 控制时间超过下午18点就时间默认显示第二天
         */
        let nowTime = TOOL.newGetDate({type: 'time'}),
          nowDate = TOOL.newGetDate({type: 'date'});
        if (TOOL.dateCompare(nowTime, TOOL.rangeTime, 'time') > 0) {
          nowDate = TOOL.changeDate(nowDate, 'd', 1)
          this.searchKeys.startTime = TOOL.newGetDate({date: nowDate, type: 'date'});
        }
      },
      
      formatDate (date) {
        return TOOL.newGetDate({date: date, type: 'formatStartupTime'})
      },
      
      /**
       * 本地存储机场查询数据
       */
      saveForm () {
        let localData = Object.assign({},this.searchKeys)
        delete localData.startTime
        appStorage.set(this.pageRouter, JSON.stringify(localData))
      },

      /**
       * 删除本地机场数据
       */
      deleteForm () {
        appStorage.remove(this.pageRouter)
      },

      /**
       * 跳到机场选择界面
       */
      jumpToAirportList () {
        this.saveForm()
        this.$router.push({
          name: 'airportList',
          query: {
            pageRouter: this.pageRouter,
            placeType: 'endPlace'
          }
        })
      },

      /**
       * 跳转到接机页面
       */
      jumpToAirportPickSearch () {
        let startPlace, endPlace;
        if(this.searchKeys.endPlace !== TOOL.placeHolder.offAirport){
          startPlace = this.searchKeys.endPlace
        }
        if(this.searchKeys.startPlace !== TOOL.placeHolder.startPlace){
          endPlace = this.searchKeys.startPlace
        }

        this.deleteForm()
        this.$router.replace({
          name:'airportPickSearch',
          query: {
            pageId:this.pageId,
            pageName:this.pageName,
            startPlace: startPlace,
            endPlace: endPlace,
            code: this.searchKeys.code
          }
        })
      },

      /**
       * 跳到城市选择界面
       */
      jumpToCitySearch (place) {
        this.saveForm()
        this.$router.push({
          name:'pageCity',
          query: {
            pageId:this.pageId,
            fromPage: this.pageRouter,
            placeType: place,
            productTypeLevelOne: 'AIRPORT_BUS'
          }
        })
      },

      /**
       * 跳转机场专线列表页面
       */
      search () {
        let validata = TOOL.formValidate([
          {value:this.searchKeys.startPlace, emptyTips:'请选择出发地'},
          {value:this.searchKeys.endPlace, emptyTips:'请选择目的地'},
          {value:this.searchKeys.startTime, emptyTips:'请选择出行时间'}
        ]);

        if(!validata.valid){
          this.$store.dispatch('showError', validata.msg);
          return false;
        }

        this.saveForm()
        this.searchKeys.startTime = TOOL.newGetDate({date: this.searchKeys.startTime, type: 'holeDate'})
        let endPlace = this.searchKeys.endPlace
        if(this.searchKeys.endPlace.indexOf('-') > -1){
          endPlace = this.searchKeys.endPlace.split('-')[0]
        }
        this.$router.push({
          name:'airportSendList',
          query:{
            pageId:this.pageId,
            startPlace: this.searchKeys.startPlace,
            endPlace: endPlace,
            startTime: this.searchKeys.startTime,
            airportCode: this.searchKeys.airportCode
          }
        });
      }
    }
  }
</script>
