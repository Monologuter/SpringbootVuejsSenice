<template>
  <div :class="[multipage === true ? 'multi-page':'single-page', 'not-menu-page', 'home-page']" style="height: 100%;width: 100%">
    <div id="areas"></div>
    <mu-card style="width: 100%; max-width: 375px; border-radius:10px;position: absolute;margin: 15px" :raised="true" v-show="scenicSearch">
      <mu-card-text style="padding: 0px">
        <mu-card style="width: 100%; max-width: 400px;border-radius:5px;top: 80px" :raised="true">
          <mu-button icon style="height: 40px;width: 40px">
            <a-icon type="appstore" style="font-size: 25px"/>
          </mu-button>
          <!--@blur="rmCardCondition"-->
          <mu-text-field style="margin: 0 ;padding: 0;vertical-align:top;height:40px;width: 275px"
                         placeholder="搜索" v-model="localName"></mu-text-field>
          <mu-button icon @click="searchAll" style="height: 40px;width: 40px">
            <a-icon type="scan" style="font-size: 20px"/>
          </mu-button>
        </mu-card>
      </mu-card-text>
    </mu-card>

    <mu-drawer :open.sync="scenicDrawer" :docked="true" :width="410">
      <mu-card style="width: 100%; max-width: 375px; border-radius:10px;position: absolute;margin: 15px" :raised="true">
        <mu-card-text style="padding: 0px">
          <mu-card style="width: 100%; max-width: 400px;border-radius:5px;" :raised="true">
            <mu-button icon style="height: 40px;width: 40px" @click="drawerHide">
              <a-icon type="poweroff" style="font-size: 20px"/>
            </mu-button>
            <!--@blur="rmCardCondition"-->
            <mu-text-field style="margin: 0 ;padding: 0;vertical-align:top;height:40px;width: 275px"
                           placeholder="搜索" v-model="localName"></mu-text-field>
            <mu-button icon @click="searchAll" style="height: 40px;width: 40px">
              <a-icon type="scan" style="font-size: 20px"/>
            </mu-button>
          </mu-card>
        </mu-card-text>
      </mu-card>
      <div style="height: 210px">
        <img :src="defaultImg" alt="" width="100%" height="210px">
      </div>

      <div v-show="scenicInfoShow" class="scenicInfo">
        <a-card :title="scenic.scenicName" :bordered="false">
          <a-tabs default-active-key="1">
            <a-tab-pane key="1" tab="基础信息">
              <ul>
                <li>地址：{{scenic.address}}</li>
                <li>热度：{{scenic.hot}}</li>
                <li>等级：{{scenic.level}}</li>
                <li>历史：{{scenic.history}}</li>
                <li>价钱：{{scenic.scenicPrice}}</li>
              </ul>
            </a-tab-pane>
            <a-tab-pane key="2" tab="路线规划">
              <a-timeline>
                <a-timeline-item v-for="(item,index) in roadData" :key="index"><div v-html="item"></div></a-timeline-item>
              </a-timeline>
            </a-tab-pane>
          </a-tabs>
        </a-card>
      </div>

      <div v-for="(scenic,i) in scenicListData" :key="i" v-show="scenicListShow">
        <a-divider><a-badge status="processing" /><i>{{scenic.address}}</i></a-divider>
        <a-card hoverable style="width: 95%;margin: 0 auto" @click="getScenicInfo(scenic)">
          <img
            v-if="scenic.webImg !== undefined && scenic.webImg.length > 0"
            :style="{backgroundImage: 'url(' + scenic.webImg + ')',height: '120px',backgroundSize: 'cover'}"
            slot="cover"
          />
          <a-card-meta :title="scenic.scenicName">
            <template slot="description" style="font-size: 10px">
              <div>
                <a-rate :default-value="parseInt((parseFloat(scenic.sold) / 1000).toFixed(1))" allow-half disabled style="float: left"/>
                <a-icon type="bug" style="float: right" @click.stop="local(scenic)" v-if="scenic.point !== ''"/>
              </div>
              <p style="clear: both;">{{scenic.history}}</p>
            </template>
          </a-card-meta>
        </a-card>
      </div>

      <div style="margin-top: 20px;font-size: 50px" v-if="scenicListData.length !== 0" v-show="scenicListShow">
        <a-card :bordered="false">
          <a-pagination :default-current="page.defaultCurrent" :total="page.total" simple @change="searchAll"/>
        </a-card>

      </div>
    </mu-drawer>
  </div>
</template>
<script>
import HeadInfo from '@/views/common/HeadInfo'
import baiduMap from '@/utils/map/baiduMap'
import scenic_map from "@/views/scenic/scenic/scenic-map"
import {mapState} from 'vuex'
import moment from 'moment'
moment.locale('zh-cn')

export default {
  name: 'HomePage',
  components: {HeadInfo, scenic_map},
  data () {
    return {
      series: [],
      chartOptions: {
        chart: {
          toolbar: {
            show: false
          }
        },
        plotOptions: {
          bar: {
            horizontal: false,
            columnWidth: '35%'
          }
        },
        dataLabels: {
          enabled: false
        },
        stroke: {
          show: true,
          width: 2,
          colors: ['transparent']
        },
        xaxis: {
          categories: []
        },
        fill: {
          opacity: 1

        }
      },
      projects: [
        {
          name: 'FEBS-Shiro',
          des: 'Spring Boot 2.0.4 & Shiro1.4.0 权限管理系统。',
          avatar: 'F'
        },
        {
          name: 'FEBS-Security',
          des: 'Spring Boot 2.0.4 & Spring Security 5.0.7 权限管理系统。',
          avatar: 'F'
        },
        {
          name: 'SpringAll',
          des: '循序渐进学习Spring Boot、Spring Cloud与Spring Security。',
          avatar: 'S'
        },
        {
          name: 'FEBS-Shiro-Vue',
          des: 'FEBS-Shiro前后端分离版本，前端架构采用Vue全家桶。',
          avatar: 'F'
        },
        {
          name: 'FEBS-Actuator',
          des: '使用Spring Boot Admin 2.0.2构建，用于监控FEBS。',
          avatar: 'F'
        }
      ],
      todayIp: '',
      todayVisitCount: '',
      totalVisitCount: '',
      userRole: '',
      userDept: '',
      lastLoginTime: '',
      welcomeMessage: '',
      localName: '',
      scenicDrawer: false,
      scenicSearch: true,
      scenicListData: [],
      defaultImg: 'static/img/default_geocode-1x.png',
      page: {
        defaultCurrent: 1,
        defaultPageSize: 10,
        total: 0,
      },
      roadData: [],
      scenic: {},
      scenicInfoShow: false,
      scenicListShow: true,
      nowPoint: {}
    }
  },
  computed: {
    ...mapState({
      multipage: state => state.setting.multipage,
      user: state => state.account.user
    }),
    avatar () {
      return `static/avatar/${this.user.avatar}`
    }
  },
  methods: {
    getRoadData() {
      let options = {
        onSearchComplete: results =>{
          if (driving.getStatus() == BMAP_STATUS_SUCCESS){
            // 获取第一条方案
            let plan = results.getPlan(0);
            // 获取方案的驾车线路
            let route = plan.getRoute(0);
            // 获取每个关键步骤,并输出到页面
            let s = [];
            for(let j = 0;j < plan.getNumRoutes(); j++){
              let route = plan.getRoute(j);
              for (let i = 0; i < route.getNumSteps(); i++){
                let step = route.getStep(i);
                s.push((i + 1) + ". " + step.getDescription());
              }
            }
            this.roadData = s
          }
        }
      };
      let driving = new BMap.DrivingRoute(baiduMap.rMap(), options);
      let start=new BMap.Point(this.nowPoint.lng,this.nowPoint.lat);
      let end=new BMap.Point(this.scenic.point.split(",")[0],this.scenic.point.split(",")[1]);
      let p1=new BMap.Point(116.460927,39.940802 );

      driving.search(start, end);
    },
    getScenicInfo(scenic) {
      this.defaultImg = scenic.webImg
      this.scenicListShow = false
      this.scenicInfoShow = true
      this.scenic = scenic
      this.local(scenic);
    },
    drawerHide() {
      this.scenicDrawer = false;
      this.scenicSearch = true
    },
    searchAll(page) {
      this.getLocal()
      this.defaultImg = 'static/img/default_geocode-1x.png'
      this.scenicListShow = true
      this.scenicInfoShow = false
      this.scenicSearch = false
      let params = {}
      if(typeof page === 'number' && page%1 === 0) {
        params.current = page
      }else {
        params.current = this.page.defaultCurrent
      }
      params.pageSize = this.page.defaultPageSize
      params.scenic = this.localName
      this.$get('/scenic/info/mapPage',{
        ...params
      }).then((r) => {
        this.scenicListData = r.data.data.records;
        this.page.total = r.data.data.total
      })
      this.scenicDrawer = true
    },
    local(scenic) {
      baiduMap.clearOverlays();
      baiduMap.rMap().enableScrollWheelZoom(true);
      let driving = new BMap.DrivingRoute(baiduMap.rMap(), {renderOptions:{map: baiduMap.rMap(), autoViewport: true}});
      console.log(JSON.stringify(this.nowPoint));
      driving.search(new BMap.Point(this.nowPoint.lng,this.nowPoint.lat), new BMap.Point(scenic.point.split(",")[0],scenic.point.split(",")[1]));
      this.getRoadData();
    },
    getLocal() {
      let geolocation = new BMap.Geolocation();
      geolocation.getCurrentPosition( r =>{
        this.nowPoint = r.point
      },{enableHighAccuracy: true})
    },
    welcome () {
      const date = new Date()
      const hour = date.getHours()
      let time = hour < 6 ? '早上好' : (hour <= 11 ? '上午好' : (hour <= 13 ? '中午好' : (hour <= 18 ? '下午好' : '晚上好')))
      let welcomeArr = [
        '喝杯咖啡休息下吧☕',
        '要不要和朋友打局LOL',
        '要不要和朋友打局王者荣耀',
        '几天没见又更好看了呢😍',
        '今天又写了几个Bug🐞呢',
        '今天在群里吹水了吗',
        '今天吃了什么好吃的呢',
        '今天您微笑了吗😊',
        '今天帮助别人解决问题了吗',
        '准备吃些什么呢',
        '周末要不要去看电影？'
      ]
      let index = Math.floor((Math.random() * welcomeArr.length))
      return `${time}，${this.user.username}，${welcomeArr[index]}`
    }
  },
  mounted () {
    baiduMap.initMap("areas");
  }
}
</script>
<style lang="less">
  #areas {
    width: 100%;
    height: 100%;
    padding: 0;
    margin: 0;
    position: absolute;
  }

  /deep/ .ant-card-body {
    padding: 10px;
  }

  /deep/ .mu-input {
    min-height: 40px;
  }

  /deep/ .scenicInfo .ant-card-head-title {
    font-size: 20px;
    font-style:italic;
    font-weight:bold
  }
  .not-menu-page {
    padding: 0;
  }

  .home-page {
    .head-info {
      margin-bottom: .5rem;
      .head-info-card {
        padding: .5rem;
        border-color: #f1f1f1;
        .head-info-avatar {
          display: inline-block;
          float: left;
          margin-right: 1rem;
          img {
            width: 5rem;
            border-radius: 2px;
          }
        }
        .head-info-count {
          display: inline-block;
          float: left;
          .head-info-welcome {
            font-size: 1.05rem;
            margin-bottom: .1rem;
          }
          .head-info-desc {
            color: rgba(0, 0, 0, 0.45);
            font-size: .8rem;
            padding: .2rem 0;
            p {
              margin-bottom: 0;
            }
          }
          .head-info-time {
            color: rgba(0, 0, 0, 0.45);
            font-size: .8rem;
            padding: .2rem 0;
          }
        }
      }
    }
    .count-info {
      .visit-count-wrapper {
        padding-left: 0 !important;
        .visit-count {
          padding: .5rem;
          border-color: #f1f1f1;
          .ant-card-body {
            padding: .5rem 1rem !important;
          }
        }
      }
      .project-wrapper {
        padding-right: 0 !important;
        .project-card {
          border: none !important;
          .ant-card-head {
            border-left: 1px solid #f1f1f1 !important;
            border-top: 1px solid #f1f1f1 !important;
            border-right: 1px solid #f1f1f1 !important;
          }
          .ant-card-body {
            padding: 0 !important;
            table {
              width: 100%;
              td {
                width: 50%;
                border: 1px solid #f1f1f1;
                padding: .6rem;
                .project-avatar-wrapper {
                  display:inline-block;
                  float:left;
                  margin-right:.7rem;
                  .project-avatar {
                    color: #42b983;
                    background-color: #d6f8b8;
                  }
                }
              }
            }
          }
          .project-detail {
            display:inline-block;
            float:left;
            text-align:left;
            width: 78%;
            .project-name {
              font-size:.9rem;
              margin-top:-2px;
              font-weight:600;
            }
            .project-desc {
              color:rgba(0, 0, 0, 0.45);
              p {
                margin-bottom:0;
                font-size:.6rem;
                white-space:normal;
              }
            }
          }
        }
      }
    }
  }
</style>

