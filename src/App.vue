<template>
  <!-- <div > -->
    <el-container id="app">
  <el-header>
    <el-alert
    title="资产配置 长期持有"
    type="error"
    center
    description="查看报告期内基金的投资策略和运作分析"
    >
  </el-alert>

  </el-header>
  <el-main>

<el-row :gutter="20" type="flex"  justify="center">
  <el-col :span="3">
    <div class="grid-content bg-purple">
    <el-card>
      <i class="el-icon-watermelon"> </i>
  <span class="card-panel-text">基金总数</span>
  <div class="card-panel-num">{{fund_count}}</div>
    </el-card>
    </div>
    </el-col>
  <el-col :span="3"><div class="grid-content bg-purple">
    <el-card>
      <i class="el-icon-potato-strips"> </i>
  <span class="card-panel-text">报告总数</span> 
  <div class="card-panel-num">{{report_count}}</div>
    </el-card>
    </div></el-col>
  <!-- <el-col :span="6"><div class="grid-content bg-purple">
    <el-card></el-card>
    </div></el-col> -->
</el-row>

<div class="block">
  <div style="margin-top: 20px;">
  
  <!-- <el-button icon="el-icon-arrow-left" slot="append" @click="preYearSearch(true)">前年</el-button> -->
   <el-date-picker
      v-model="report_year"
      @change="search"
      type="year"
      placeholder="选择年份"
      value-format="yyyy"
      :picker-options="pickerOptions">
      >
  </el-date-picker>
  <!-- <el-button slot="append" @click="preYearSearch(false)">后年<i class="el-icon-arrow-right el-icon--right"></i></el-button> -->

  <el-autocomplete
  popper-class="my-autocomplete"
  placeholder="请输入基金名称或编号           " 
  v-model="code"
   :trigger-on-focus="false"
  :fetch-suggestions="querySearch"
  class="input"
  @select="handleSelect"
  >  
 <template slot-scope="props">
    <div class="fund_name">{{ props.item.fund_name }}</div>
    <span class="fund_code">{{ props.item.fund_code }}</span>
  </template>
  </el-autocomplete>

   <el-button slot="append" type="primary" icon="el-icon-search" :loading="false" @click="search">搜索</el-button>
  </div>

<el-row :gutter="20">
  
  <el-col :span="7" v-for="(item,index) in fund_report_list" :offset="1" :key="index">
    <el-card shadow="hover">
      <div slot="header" class="clearfix">
        <span>{{item.report_name}}</span>
      </div>
        <div class="bottom clearfix">
           <p>{{item.report_content}}</p> 
          <time class="time">{{ item.create_time }}</time>
          <el-button type="text" class="button" @click="clickFundStockDetail(item)">查看基金持仓</el-button>
<el-dialog title="前十名股票投资明细" width="60%" :visible.sync="dialogTableVisible" @open="open()">
  <el-table :data="gridData">
    <el-table-column property="rank_num" label="序号" width="50"></el-table-column>
    <el-table-column property="stock_code" label="股票代码" width="100"></el-table-column>
    <el-table-column property="stock_name" label="股票名称"></el-table-column>
    <el-table-column property="stock_nums" label="数量（股）"></el-table-column>
    <el-table-column property="stock_value" label="公允价值（元）"></el-table-column>
    <el-table-column property="ratio" label="占基金资产净值比例（%）"></el-table-column>
  </el-table>
</el-dialog>
        </div>
     
    </el-card>
  </el-col>
  
</el-row>
</div>
<!-- <img src="./assets/logo.png"> -->
    <!-- <img src="./pic/houying.jpg" height="500px"> -->
  </el-main>

  <el-footer>
      <div style="width:300px;margin:0 auto; padding:20px 0;">
		 		<a target="_blank" href="http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11010502047213" style="display:inline-block;text-decoration:none;height:20px;line-height:20px;"><img src="./pic/beian.png" style="float:left;"/><p style="float:left;height:20px;line-height:20px;margin: 0px 0px 0px 5px; color:#939393;">京公网安备 11010502047213号</p></a>
        <el-link href="https://beian.miit.gov.cn/" type="info"> 京ICP备2021030215号-1</el-link> 
		 	</div>
    </el-footer>
  </el-container>


<!-- </div> -->

  

</template>
<script>
export default {

  data(){
    let showDate =new Date();
    let year = showDate.getFullYear();
    return {
      code:'166002',
      season:'',
      fund_report:'',
      report_year:''+year,
      fund_report_list:[],
      fund_list:[],
      fund_count:"",
      report_count:'',
      gridData: [],
      dialogTableVisible: false,
      pickerOptions: {
          disabledDate(time) {
            return time.getTime() > Date.now();
          }
          // },
          // shortcuts: [{
          //   text: '前年',
          //   onClick(picker) {
          //     let date = new Date()
          //     console.log(this.report_year)
          //     if(this.report_year != undefined){
          //       date = new Date(this.report_year)
          //     }
          //     // this.report_year = date.getFullYear()+'';
          //     date.setTime(date.getTime() - 3600 * 1000 * 24 * 365);
          //     this.report_year = date.getFullYear()+'';
          //     console.log(this.report_year)
          //     picker.$emit('pick',new Date(this.report_year));
          //   }
          // }]
        },
    }
  },

    // watch:{
    //   code: function (val) {
    //   //this.code = val + ' '
    //   alert(val)
    // }
    // },

  methods: {
    startHacking () {
      this.$notify({
        title: 'It works!',
        type: 'success',
        message: 'We\'ve laid the ground work for you. It\'s time for you to build something epic!',
        duration: 5000
      })
    },
    search(){
      if(!this.code || this.code =='' || !this.report_year || this.report_year == ''){
        this.$message({
          message: '请输入年份和基金代码！',
          type: 'error'
        });
        return
      }
     
      const url = "http://82.156.11.109:5000/api/query_fund_report?code="+this.code.split(" ")[0]+"&year="+this.report_year
      // const url = "http://localhost:5000/api/query_fund_report?code="+this.code.split(" ")[0]+"&year="+this.report_year
      this.$axios.get(url).then((res) =>{
        const result = res.data.list;
        if(result == undefined || result.length <= 0){
          this.$message({
           message: '暂未收录该基金报告',
           });
          return
        }
        this.fund_report_list = result
      })
    },
    preYearSearch(preyear_flag){
      console.log("preYearSearch")
      console.log(preyear_flag)
      if(!this.code || this.code =='' || !this.report_year || this.report_year == ''){
        this.$message({
          message: '请输入年份和基金代码！',
          type: 'error'
        });
        return
      }
     
     console.log(this.report_year-1)
     var pre_year = parseInt(this.report_year)-1
     if(!preyear_flag){
       console.log("next year")
       pre_year = parseInt(this.report_year)+1
     }
     //todo 更改日期框中的显示
     this.report_year = ''+pre_year
      const url = "http://82.156.11.109:5000/api/query_fund_report?code="+this.code.split(" ")[0]+"&year="+pre_year
      // const url = "http://localhost:5000/api/query_fund_report?code="+this.code.split(" ")[0]+"&year="+this.report_year
      this.$axios.get(url).then((res) =>{
        const result = res.data.list;
        if(result == undefined || result.length <= 0){
          this.$message({
          message: '暂未收录该基金报告',
           });
          return
        }
        this.fund_report_list = result
      })
    },
    // 搜索建议
    querySearch(queryString, cb) {
      const url = "http://82.156.11.109:5000/api/query_fund?name="+this.code
      this.$axios.get(url).then((res) =>{
        this.fund_list = res.data.list;
        cb(res.data.list)
      })
      },

      createFilter(queryString) {
        return (fund) => {
          return (fund.fund_name.toLowerCase().indexOf(queryString.toLowerCase()) === 0);
        };
      },

      handleSelect(item) {
        this.code=item.fund_code + " " + item.fund_name;
      },

    geCount() {
      const url = "http://82.156.11.109:5000/api/get_count"
      this.$axios.get(url).then((res) =>{
        this.fund_count = res.data.list[0].fund_count;
        this.report_count = res.data.list[1].report_count;
      })
      },
      validCode(item){
        console.log("验证")
      },
    clickFundStockDetail(item){
        this.dialogTableVisible = true;
        console.log(item.season);
        this.season = item.season;
      },
    open() {
      const url = "http://82.156.11.109:5000/api/query_fund_stock_detail?code="+this.code.split(" ")[0]+"&year="+this.report_year+"&season="+this.season
      this.$axios.get(url).then((res) =>{
        this.gridData=res.data.list;
      })
      },


  },
   mounted() {
     console.log("页面加载啦")
     this.geCount();
     this.search();

    }
}
</script>

<style>
#app {
  /* font-family: Helvetica, sans-serif; */
  font-family: "Microsoft YaHei","Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","微软雅黑",Arial,sans-serif;
  text-align: center;
  font-size: small;
}

.time {
    font-size: 13px;
    color: #999;
  }

  .box-card {
    width: 480px;
  }
  
  /* .bottom {
    margin-top: 13px;
    line-height: 12px;
  } */

  .button {
    padding: 0;
    float: right;
  }

  .image {
    width: 100%;
    display: block;
  }

  .clearfix:before,
  .clearfix:after {
      display: table;
      content: "";
  }

  .clearfix:after {
      clear: both
  }

  .el-row {
    margin-bottom: 20px;
    display:flex;
    flex-wrap: wrap;
  }
.el-row  .el-card {
    min-width: 100%;
    height: 100%;
    margin-right: 20px;
    margin-top: 10px;
    transition: all .5s;
  }

.el-col-offset-1{
margin-left: 1.16667%;
    margin-top: 1%;
}

.el-input-group {
  width:50%
}

.el-col-8 {
    width: 30.33333%;
}

.el-header, .el-footer {
    background-color: #fff;
    color: #333;
    text-align: center;
    /* line-height: 20px; */
  }

.el-autocomplete{
  /* display: */
  /* display: block; */
  width:50%;
}

.my-autocomplete input{
  widows: 150%;
}

  .my-autocomplete li{
    line-height: normal;
    padding: 7px;
}

.my-autocomplete li .fund_name{
      text-overflow: ellipsis;
      overflow: hidden;
}

.my-autocomplete li .fund_code{
   font-size: 12px;
   color: #b4b4b4;
}

.my-autocomplete li .highlighted{
  color: #ddd;
}

.my-autocomplete li .fund_code{
  color: #ddd;
}

.card-panel-text{
    line-height: 18px;
    color: rgba(0,0,0,.45);
    font-size: 16px;
    margin-bottom: 12px;
}

.card-panel-num{
  font-size: 20px;
}

</style>