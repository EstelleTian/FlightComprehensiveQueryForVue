<template>
  <el-container>
    <el-header height="60px">航班方向信息检测查询</el-header>
    <el-container class="content">
      <el-aside width="240px">
        <el-menu
          default-active="1"
        >
          <el-menu-item index="1">
            <i class="el-icon-search"></i>
            <span slot="title">未命中方向查询</span>
          </el-menu-item>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view/>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
  export default {
    name: 'Layout',
    data () {
      return {
      }
    }
  }
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .el-container{
    height: 100vh;
  }
  .el-container.content{
    display: flex;
    display: -webkit-flex;
    display: -o-flex;
  }
  .el-header{
    background-color: #B3C0D1;
    color: #333;
    text-align: center;
    line-height: 60px;
    font-size: 2rem;
  }

  .el-aside {
    background-color: #D3DCE6;
    color: #333;
    text-align: center;
    /*line-height: 200px;*/
  }

  .el-main {
    background-color: #E9EEF3;
    color: #333;
    text-align: center;
    flex: 1;
    -webkit-flex: 1;
    -o-flex: 1;
    padding: 10px 20px;
  }
</style>
