<template>
  <div id="start">

    <!--导航-->
    <el-breadcrumb separator="/" class="breadcrumb">
      <el-breadcrumb-item :to="{ name:'start' }">用户基础信息</el-breadcrumb-item>
      <el-breadcrumb-item>用户状态统计</el-breadcrumb-item>
    </el-breadcrumb>

    <!--筛选-->
    <div class="warp_filter">
      <el-alert
        title="数据说明"
        type="info"
        description="本页面的统计数字为每天凌晨统计一次,当天产生的新数据将于第二天凌晨统计，但由于用户状态的特殊性，点击数量之后的详情是实时的。" style="margin-bottom: 10px">
      </el-alert>
      <template>
        <el-select v-model="filter.channels" placeholder="切换渠道" @change="valueChange">
          <el-option
            v-for="(item,index) in versions.app_channels"
            :label="item.name"
            :value="item.name" :key="index"
          >
          </el-option>
        </el-select>
      </template>
      <template>
        <el-select v-model="filter.versions" placeholder="版本筛选" @change="valueChange">
          <el-option
            v-for="(item,index) in versions.app_versions"
            :label="item.name"
            :value="item.name" :key="index">
          </el-option>
        </el-select>
      </template>
      <!--展示-->
      <el-row :gutter="20" class="">
        <el-col :span="12">
          <span style="margin: 20px 0;display: inline-block"><i class="el-icon-menu" style="margin-right: 10px"></i>用户状态统计饼图</span>
          <div id="main" style="width: 99%;height: 500px;border: 1px solid #D3DCE6">

          </div>
        </el-col>
        <el-col :span="12">
          <span style="margin: 20px 0;display: inline-block"><i class="el-icon-menu" style="margin-right: 10px"></i>用户状态统计列表</span>
          <table class="t1">
            <tbody class="table">
            <tr>
              <th>用户状态</th>
              <th>数量</th>
              <th>占比</th>
            </tr>
            <tr>
              <td>正常</td>
              <td>{{tableData[0] ? tableData[0].total : 0}}</td>
              <td>{{tableData[0] ? tableData[0].percent : 0}}%</td>
            </tr>
            <tr>
              <td>封号</td>
              <td><span style="cursor: pointer;background-color: #333;padding: 1px 3px;color: #fff"
                        @click="details">{{tableData[1] ? tableData[1].total : 0}}</span></td>
              <td>{{tableData[1] ? tableData[1].percent : 0}}%</td>
            </tr>
            </tbody>
          </table>
        </el-col>
      </el-row>

      <!--数量弹窗-->
      <el-dialog v-model="dialogTableVisible" :title="pageInfo.is.is_enabled">
        <el-alert
          title="数据说明"
          type="info"
          description="封号详情的数据是实时的，与封号数量对应不上属正常情况。" style="margin-bottom: 10px">
        </el-alert>
        <el-table :data="dialogData" style="text-align: center">
          <el-table-column label="被封账号" width="150">
            <template scope="scope">
            <span class="dialog_num"
                  @click="userInfo(scope.row)">{{scope.row.username}}</span>
            </template>
          </el-table-column>
          <el-table-column property="admin_username" label="封号人"></el-table-column>
          <el-table-column label="封号时间" width="200">
            <template scope="scope">
              <span>{{scope.row.created_at | Time}}</span>
            </template>
          </el-table-column>
        </el-table>
        <div slot="footer">
          <el-pagination
            @current-change="handleCurrentChange"
            :current-page="currentPage"
            :page-size="pageSize"
            layout="total, prev, pager, next, jumper"
            :total="totalSize"
            class="page">
          </el-pagination>
        </div>
      </el-dialog>

    </div>

    <user-detail :visab="dialogVisible" :name="username" @closeDialog="cdialog"></user-detail>

  </div>
</template>

<script>
  import * as API from '../../../api/api'
  import userDetail from '../../publicView/accoutInfo/index.vue'
  import {mapGetters, mapActions} from 'vuex'
  export default {
    data(){
      return {
        /*筛选*/
        filter: {
          versions: '',
          channels: '',
        },
        /*饼状图*/
        options: {
          title: {
            text: null
          },
          tooltip: {
            pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
          },
          plotOptions: {
            pie: {
              allowPointSelect: true,
              cursor: 'pointer',
              dataLabels: {
                enabled: true,
              },
              showInLegend: true
            }
          },
          series: [{
            type: 'pie',
            name: '用户状态',
            data: [
              ['a', 10],
              ['b', 20],
              ['c', 30],
              ['d', 40]
            ]
          }],
          credits: false
        },
        /*列表*/
        tableData: [],

        /*弹窗*/
        dialogTableVisible: false,
        dialogData: [],
        pageInfo: {
          is: '',
          page: '',
          device_model: ''
        },
        dialogVisible: false,
        username: '姓名',
        /*分页*/
        val: {},
        currentPage: 1,
        totalSize: 0,
        pageSize: 15

      }
    },
    computed: {
      ...mapGetters(['versions']),
    },
    components: {
      userDetail
    },
    methods: {
      ...mapActions({
        ud_base: 'UD_base_info',
      }),
      cdialog(){
        this.dialogVisible = false
      },
      userInfo(row){
        this.username = row.username
        this.dialogVisible = true;
        window.sessionStorage.setItem('userId', row.user_id)
        this.ud_base({limit: 10})
      },
      //图表数据转为数组
      line(json){
        var arr = [];
        for (var i = 0; i < json.length; i++) {
          if (json[i]['is_enabled'] == true) {
            json[i]['is_enabled'] = '正常'
          } else {
            json[i]['is_enabled'] = '封号'
          }
          arr.push([json[i]['is_enabled'], parseFloat(json[i].percent)])
        }
        return arr
      },

      /*筛选*/
      valueChange(){
        this.getInfo({app_version: this.filter.versions, app_channel: this.filter.channels}).then(res => {
          this.tableData = res.data.data.items;
          this.options.series[0].data = this.line(res.data.data.items);
          this.$HighCharts.chart('main', this.options);
        })
      },

      /*列表*/
      //获取用户状态统计信息
      getInfo(parm){
        var p = new Promise((resolve, reject) => {
          const token = JSON.parse(window.sessionStorage.getItem('loginInfo')).token;
          this.$http({
            method: 'GET',
            url: API.status_stat,
            headers: {'Authorization': token},
            params: parm
          }).then(function (res) {

            if (res.status == 200) {
              resolve(res)
            } else {
              reject(res)
            }

          })
        })
        return p;
      },
      //过滤用户状态
      filterStar(data){
        return data == true ? '正常' : '封号';
      },

      /*弹窗*/
      /*查看详情*/
      details(){
        this.dialogTableVisible = true;
        var options = {
          page: 1,
          limit: this.pageSize,
          is_enabled: false,
          app_version: this.filter.versions,
          app_channel: this.filter.channels
        }
        this.getDetails(options).then(res => {
          if (res.status == 200) {
            let data = res.data.data
            this.dialogData = data.logs;
            this.currentPage = data.current_page;
            this.totalSize = data.total_count
          }
        });
      },
      //获取统计详情
      getDetails(parm){
        return new Promise((resolve, reject) => {
          const token = JSON.parse(window.sessionStorage.getItem('loginInfo')).token;
          this.$http({
            method: 'GET',
            url: API.status_stat_details,
            headers: {'Authorization': token},
            params: parm
          }).then(function (res) {
            if (res.status == 200) {
              resolve(res)
            }
          }).catch(function (err) {
            reject(err)
          })
        })
      },

      /*分页*/
      handleCurrentChange(val){
        this.currentPage = val;
        this.getDetails({
          page: val, limit: this.pageSize, app_version: this.filter.versions,
          app_channel: this.filter.channels
        })
      }

    },
    mounted(){
      //绘制图形
      this.getInfo().then(res => {
        this.tableData = res.data.data.items;
        this.options.series[0].data = this.line(res.data.data.items);
        this.$HighCharts.chart('main', this.options);
      })
    },
  }
</script>

<style scoped>
  #start {
    padding: 10px;
  }

  /*导航*/
  #userInfo .breadcrumb {
    height: 30px;
    line-height: 30px;
  }

  .warp_filter {
    text-align: left;
    padding: 10px;
    background-color: #fff;
    border: 1px solid #D3DCE6;
    margin-top: 20px;
  }

  .el-select {
    width: 200px;
  }

  .el-input {
    width: 200px;
  }
</style>
