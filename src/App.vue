<template>
  <div id="app">
    <el-container>
  <el-header>
    <el-alert
    title="查看报告期内基金的投资策略和运作分析"
    type="success"
    center
    description="这是一句绕口令：黑灰化肥会挥发发灰黑化肥挥发；灰黑化肥会挥发发黑灰化肥发挥。 黑灰化肥会挥发发灰黑化肥黑灰挥发化为灰……"
    show-icon
    >
  </el-alert>

  </el-header>
  <el-main>

<el-row :gutter="20" type="flex"  justify="center">
  <el-col :span="3"><div class="grid-content bg-purple">
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

   <el-date-picker
      v-model="report_year"
      type="year"
      placeholder="选择年份"
      value-format="yyyy">
  </el-date-picker>

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
  <el-col :span="8" v-for="(item,index) in fund_report_list" :offset="1" :key="index">
    <el-card shadow="hover">
      <div slot="header" class="clearfix">
        <span>{{item.report_name}}</span>
      </div>
        <div class="bottom clearfix">
           <p>{{item.report_content}}</p>
          <time class="time">{{ item.create_time }}</time>
          <el-button type="text" class="button">查看基金持仓</el-button>
        </div>
     
    </el-card>
  </el-col>
</el-row>
</div>
<!-- <img src="./assets/logo.png"> -->
    <!-- <img src="./pic/houying.jpg" height="500px"> -->
  </el-main>
  </el-container>


</div>

  

</template>
<script>
export default {

  data(){
    let showDate =new Date();
    let year = showDate.getFullYear();
    return {
      code:'',
      fund_report:'',
      report_year:''+year,
      fund_report_list:[],
      fund_list:[],
      fund_count:"",
      report_count:''
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
      }
  },
   mounted() {
     console.log("页面加载啦")
     this.geCount();

    }
}
</script>

<style>
#app {
  font-family: Helvetica, sans-serif;
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