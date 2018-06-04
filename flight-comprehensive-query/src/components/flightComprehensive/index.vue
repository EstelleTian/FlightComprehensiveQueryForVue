<template>
  <div>
    <el-form :inline="true" :model="formInline" :rules="rules" ref="directionForm" class="form-inline">
      <el-form-item
        label="机场"
        prop="airport"
      >
        <el-input v-model="formInline.airport"  placeholder="请输入机场，如ZU,ZW"  class="search-airport"></el-input>
      </el-form-item>
      <el-form-item
        label="起止时间"
        prop="dateRange"
      >
        <el-date-picker
          class="search-date"
          v-model="formInline.dateRange"
          type="datetimerange"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-radio v-model="formInline.hasFPL" label="1">有FPL</el-radio>
        <el-radio v-model="formInline.hasFPL" label="0">无FPL</el-radio>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          size="medium"
          :loading="isLoading"
          @click="searchFlights('directionForm')"
        >查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-input placeholder="快速过滤" v-model="quicklySearch" class="input-with-select" @change="filterTable">
          <el-button slot="append" icon="el-icon-search"></el-button>
        </el-input>
      </el-form-item>
    </el-form>
    <el-form :inline="true" class="form-inline-res" v-if="resultShow">
      <el-form-item
        label="机场:"
        prop="airport"
      >
        {{searchRes.airport}}
      </el-form-item>
      <el-form-item
        label="起止时间:"
        prop="dateRange"
      >
        {{searchRes.start}} 至 {{searchRes.start}}
      </el-form-item>
      <el-form-item
        label="有无FLP报:"
      >
        {{searchRes.hasFPL}}
      </el-form-item>
      <el-form-item
        label="数据生成时间:"
      >
        {{searchRes.generateTime}}
      </el-form-item>
    </el-form>
    <el-table
      ref="flightTable"
      :data="tableData.slice( (currentPage -1 )*pageSize, currentPage*pageSize)"
      border
      :default-sort="{prop: 'executedate', order: 'descending'}"
      style="width: 100%">
      <el-table-column
        fixed
        align="center"
        type="index"
        width="50">
      </el-table-column>
      <el-table-column
        fixed
        align="center"
        sortable
        prop="flightId"
        label="航班号"
      >
      </el-table-column>
      <el-table-column
        align="center"
        sortable
        prop="depap"
        label="起飞机场"
      >
      </el-table-column>
      <el-table-column
        align="center"
        sortable
        prop="arrap"
        label="降落机场"
      >
      </el-table-column>
      <el-table-column
        align="center"
        sortable
        prop="sobt"
        label="SOBT"
        :formatter="formatterDateTime"
      >
      </el-table-column>
      <el-table-column
        align="center"
        sortable
        prop="eobt"
        label="EOBT"
        :formatter="formatterDateTime"
      >
      </el-table-column>
      <el-table-column
        align="center"
        sortable
        prop="atd"
        label="ATOT"
        :formatter="formatterDateTime"
      >
      </el-table-column>
      <el-table-column
        align="center"
        sortable
        prop="executedate"
        label="EXECUDATE"
        :formatter="formatterDate"
      >
      </el-table-column>
    </el-table>
    <div class="block">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[ 10, 20, 30]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </div>
  </div>

</template>
<script>
  export default {
    data() {
      let validateAirport = (rule, value, callback)=>{
        const reg = /^[a-zA-Z]{1,4}$/g;
        if(value === ''){
           callback(new Error('机场不能为空'));
        }else if( !reg.test(value) ){
          callback(new Error('请输入1-4位机场代码'));
        }else{
          callback();
        }
      };
      return {
        formInline:{
          airport: 'zwww',
          dateRange: [],
          hasFPL: '1',
        },
        searchRes: {
          airport: '',
          start: '',
          end: '',
          hasFPL: '',
          generateTime: ''
        },
        isLoading: false,
        quicklySearch: '',
        resultShow: false,
        rules: {
          airport: [
            { required: true, message: '请输入机场' },
            { validator: validateAirport}
          ],
          dateRange: [
            { required: true, message: '请输入起止时间' }
          ]
        },
        tableData: [],
        total: 0,
        currentPage: 1,
        pageSize: 20,
        tableDataMap: {}
      }
    },
    created(){
      this.initDate();
     },
    methods: {
      initDate(){
        const now = new Date();
        const year = now.getFullYear();
        const month = now.getMonth();
        const day = now.getDate();
        let defaultArr = [];
        let start = new Date(year, month, day, 0, 0);
        let end = new Date(year, month, day, 0, 59);
        defaultArr.push(start);
        defaultArr.push(end);
        this.formInline.dateRange = defaultArr;
      },
      convertDate(date){
        let getTwoPos = (num) => {
          num = num * 1;
          if(num < 10){
            num = '0' + num;
          }
          return num + '';
        }
        let y = date.getFullYear();
        let m = date.getMonth()+1;
        m = getTwoPos(m);
        let d = date.getDate();
        d = getTwoPos(d);
        let hour = date.getHours();
        hour = getTwoPos(hour);
        let mins = date.getMinutes();
        mins = getTwoPos(mins);
        return y + m + d + hour + mins;
      },
      formatterDateString( dateStr ){
        if( dateStr == '' || undefined == dateStr || dateStr.length < 12){
          return '';
        }
        const year =  dateStr.substring(0,4);
        const month =  dateStr.substring(4,6);
        const day =  dateStr.substring(6,8);
        const hour =  dateStr.substring(8,10);
        const mins =  dateStr.substring(10,12);
        return year + '-' + month + '-' + day + ' ' + hour + ':' + mins;
      },
      formatterDateTime(row, column, cellValue, index){
        if(cellValue){
          return this.formatterDateString(cellValue);
        }
        return cellValue;
      },
      formatterDate(row, column, cellValue, index){
        if(cellValue){
          const year =  cellValue.substring(0,4);
          const month =  cellValue.substring(4,6);
          const day =  cellValue.substring(6,8);
          return year + '-' + month + '-' + day;
        }
        return cellValue;


      },
      convertTotableData(resObj){
        let tableData = [];
        for(var index in resObj){
          const flight = resObj[index];
          tableData.push(flight);
        }
        return tableData;

      },
      searchFlights(formName){
        let _this = this;
        _this.$refs[formName].validate((valid)=>{
          if(valid){
            _this.isLoading = true;
            //发送请求
            const url = 'http://192.168.243.187:38180/Flights/retrieve';
            const airport = this.formInline.airport;
            const start = this.convertDate(this.formInline.dateRange[0]);
            const end = this.convertDate(this.formInline.dateRange[1]);
            const hasFPL = this.formInline.hasFPL*1 ? true : false;
              let params = {
              airport,
              start,
              end,
              hasFPL
            }
            _this.$axios.request({
              url: url,
              params: params,
            }).then((res)=>{
              const data = res.data;
              const generateTime = data.generateTime;
              const result = data.result;
              const tableDataArr = _this.convertTotableData(result);
              _this.tableDataMap = result;
              _this.tableData = tableDataArr;
              _this.total = tableDataArr.length;
              _this.searchRes.generateTime = _this.formatterDateString( generateTime );
              _this.searchRes.airport = airport.toUpperCase();
              _this.searchRes.start = _this.formatterDateString( start );
              _this.searchRes.end = _this.formatterDateString( end );
              _this.searchRes.hasFPL = hasFPL ? '有' : '无';
              _this.resultShow = true;
              _this.isLoading = false;
            }).catch(function (error) {
              console.log(error);
              _this.isLoading = false;
            });
          }
        })


      },
      filterTable(){
         const searchValue = this.quicklySearch;
         let tableFliterArr = [];
         const tableDataMap = this.tableDataMap;
         for(let i in tableDataMap){
            const flight = tableDataMap[i];
            const flightId = flight.flightId.toLowerCase() || "";
            const flightIdFlag = ( flightId.indexOf(searchValue) > -1 );
            const depap = flight.depap || "";
            const depapFlag = ( depap.toLowerCase().indexOf(searchValue) > -1 );
            const arrap = flight.arrap || "";
            const arrapFlag = ( arrap.toLowerCase().indexOf(searchValue) > -1 );
            const sobt = flight.sobt || "";
            const sobtFlag = ( sobt.toLowerCase().indexOf(searchValue) > -1 );
            const eobt = flight.eobt || "";
            const eobtFlag = ( eobt.toLowerCase().indexOf(searchValue) > -1 );
            const atot = flight.atd || "";
            const atotFlag = ( atot.toLowerCase().indexOf(searchValue) > -1 );
            const executedate = flight.executedate || "";
            const executedateFlag = ( executedate.toLowerCase().indexOf(searchValue) > -1 );
            if( flightIdFlag || depapFlag || arrapFlag || sobtFlag || eobtFlag || atotFlag || executedateFlag ){
              tableFliterArr.push(flight);
            }
         }
         this.tableData = tableFliterArr;
      },
      handleSizeChange(val) {
        this.pageSize = val;
        console.log(`每页 ${val} 条`);
      },
      handleCurrentChange(val) {
        this.currentPage = val;
        console.log(`当前页: ${val}`);
      }
    }
  }
</script>
<style>
  .search-airport{
    width: 16rem;
  }
  .search-airport input{
    text-transform: uppercase;
  }
  .form-search{
    margin-bottom: 10px;
  }
  .search-radio{
    line-height: 40px;
  }
  .el-form-item {
    margin-bottom: 5px;
  }
  .form-inline-res .el-form-item__label{
    font-size: 16px;
  }
  .form-inline-res .el-form-item__content{
    font-size: 16px;
    font-weight: bold;
    margin-right: 20px;
  }
  .el-pagination{
    display: inline-block;
    float: right;
  }
  .el-table td, .el-table th{
    padding: 6px 0;
  }
</style>
