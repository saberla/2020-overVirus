<template>
  <div class="home"
    v-loading="loading"
    element-loading-text="当前api访问量大，请耐心等待"
    element-loading-spinner="el-icon-loading"
  >
    <el-row>
      <el-col :span="24" class="title"><span class="title">2020-virus 新型冠状病毒</span></el-col>
    </el-row>
    <el-row class="announceContainer">
      <el-col :span="24">
        <el-button type="primary" @click="dialogVisible = true" size="small">声明</el-button>
        <el-dialog
          title="声明"
          :visible.sync="dialogVisible"
          width="80%"
          center>
          <div class="announce">
            <p>数据来源：<a href="http://www.dxy.cn/">丁香园</a>(与抖音百度数据有微小差错)</p>
            <p>爬取数据来源：<a href="https://github.com/BlankerL/DXY-2019-nCoV-Crawler">BlankerL</a></p>
            <p style="color:red">武汉加油！中国加油！</p>
            <p>若页面数据无法正常显示，请刷新一下</p>
            <p>written by 蒲智鹏</p>
          </div>
          <span slot="footer" class="dialog-footer">
            <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
          </span>
        </el-dialog>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <p style="color:rgb(139, 139, 139)">数据更新时间：{{updateTime}}</p>
      </el-col>
    </el-row>
    <div class="overview">
      <el-row>
        <el-col :span="6">
          <div>
            <p style="color:red;font-size:26px;font-weight: bolder">{{overData.confirmedCount}}</p>
            <p style="font-size:9px">确诊</p>
            <p style="font-size:9px">较昨日<span style="color:red">+{{overData.confirmedIncr}}</span></p>
          </div>
        </el-col>
        <el-col :span="6">
          <div>
            <p style="color:orange;font-size:26px;font-weight: bolder">{{overData.suspectedCount}}</p>
            <p style="font-size:9px">疑似</p>
            <p style="font-size:9px">较昨日<span style="color:orange">+{{overData.suspectedIncr}}</span></p>
          </div>
        </el-col>
        <el-col :span="6">
          <div>
            <p style="color:rgb(132, 202, 28);font-size:26px;font-weight: bolder">{{overData.curedCount}}</p>
            <p style="font-size:9px">治愈</p>
            <p style="font-size:9px">较昨日<span style="color:rgb(132, 202, 28)">+{{overData.curedIncr}}</span></p>
          </div>
        </el-col>
        <el-col :span="6">
          <div>
            <p style="color:black;font-size:26px;font-weight: bolder">{{overData.deadCount}}</p>
            <p style="font-size:9px">死亡</p>
            <p style="font-size:9px">较昨日<span style="color:black">+{{overData.deadIncr}}</span></p>
          </div>
        </el-col>
      </el-row>
    </div>
    <div class="img" style="margin-top:10px" width='100%'>
      <img :src="imgUrl" alt="趋势图" width="100%">
    </div>
    <div class="chinaTableContainer">
      <el-row>
        <el-col :span="24">
          <p style="text-align:center;margin-top:14px;font-size:20px">国内</p>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24">
          <el-table
            :data="ChinaData"
            :default-sort = "{prop: 'confirmedCount', order: 'descending'}"
            style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-table
                  :data="scope.row.cities"
                  style="width: 100%">
                  <el-table-column
                    prop="cityName"
                    label="城市">
                  </el-table-column>
                  <el-table-column
                    prop="confirmedCount"
                    label="确诊">
                  </el-table-column>
                  <el-table-column
                    prop="deadCount"
                    label="死亡">
                  </el-table-column>
                  <el-table-column
                    prop="curedCount"
                    label="治愈">
                  </el-table-column>
                </el-table>
              </template>
            </el-table-column>
            <el-table-column
              label="省份"
              prop="provinceName">
            </el-table-column>
            <el-table-column
              label="确诊"
              sortable
              prop="confirmedCount">
            </el-table-column>
            <el-table-column
              label="死亡"
              prop="deadCount">
            </el-table-column>
            <el-table-column
              label="治愈"
              prop="curedCount">
            </el-table-column>
          </el-table>
        </el-col>
      </el-row>
    </div>
    <div class="otherTableContainer">
      <el-row>
        <el-col :span="24">
          <p style="text-align:center;margin-top:14px;font-size:20px">国外</p>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24">
          <el-table
            :data="OtherData"
            style="width: 100%">
            <el-table-column
              prop="country"
              label="国家">
            </el-table-column>
            <el-table-column
              prop="confirmedCount"
              label="确诊">
            </el-table-column>
            <el-table-column
              prop="deadCount"
              label="死亡">
            </el-table-column>
            <el-table-column
              prop="curedCount"
              label="治愈">
            </el-table-column>
          </el-table>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
export default {
  name: 'home',
  data () {
    return {
      dialogVisible: false,
      imgUrl: '',
      loading: true,
      updateTime: '',
      overData: {},
      ChinaData: [],
      OtherData: []
    }
  },

  created () {
    this.getOverAll()
    this.getWholeCountry()
  },

  methods: {
    // 获取总览数据
    getOverAll () {
      this.$axios.get('https://lab.isaaclin.cn/nCoV/api/overall').then(res => {
        this.loading = false
        this.overData = res.data.results[0]
        this.imgUrl = res.data.results[0].dailyPics[0]
        let date = new Date(res.data.results[0].updateTime)
        this.updateTime = date.getFullYear() + '-' + (date.getMonth() + 1) + '-' + date.getDate() + ' ' + date.getHours() + ':' + date.getMinutes() + ':' + date.getSeconds()
      }).catch(err => {
        this.loading = false
        console.log('发生错误：', err)
      })
    },
    getWholeCountry () {
      this.$axios.get('https://lab.isaaclin.cn//nCoV/api/area').then(res => {
        this.laoding = false
        console.log(res)
        for (let i in res.data.results) {
          if (res.data.results[i].country === '中国') {
            this.ChinaData.push(res.data.results[i])
          } else {
            this.OtherData.push(res.data.results[i])
          }
        }
      }).catch(err => {
        this.loading = false
        console.log('发生错误：', err)
      })
    }
  }
}
</script>

<style lang="less" scoped>
.home{
  .title{
    text-align: center;
    font-size: 21px;
  }
  .announceContainer{
    margin-top: 20px;
    text-align: center;
  }
  .announce{
    text-align: center;
  }
  .overview{
    text-align: center;
    padding-left: 10px;
    p{
      margin: 5px;
    }
  }
}
</style>
<style lang="less">
// 声明弹出框边距
.el-dialog--center .el-dialog__body{
  padding: 12px !important;
}
.el-dialog--center .el-dialog__header{
  padding: 0 !important;
  padding-top: 12px !important;
}
.el-dialog--center .el-dialog__footer{
  padding: 0 !important;
  padding-bottom: 12px !important;
}
// 表格边距
.el-table td, .el-table th{
  padding: 0;
  height: 48px;
  width: 100%;
}
</style>
