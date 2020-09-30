<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
     <Table
      :tableConfig="tableConfig"
      :tableList="tableList"
      :showIndex="false"
      :border="false"
      :showPagination="true"
      :headerCellStyle="{ background: '#f2f2f2', color: '#606266' }"
      @pagination="paginationChange"
      :paginationData="paginationData"
    >
      <template #buttons>
        <el-table-column align="center" label="操作">
          <template slot-scope="scope">
            <template>
              <el-button>上传记录</el-button>
              <el-button v-if="scope.row.status == 1" size="mini" type="text" @click.native="handDetail(scope.row)">查看</el-button>
            </template>
          </template>
        </el-table-column>
      </template>
    </Table>
  </div>
</template>

<script>
import Table from "./Table";

export default {
   components: {
    Table
  },
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
       paginationData: {
        size: 10,
        current: 1,
        total: 1
      },
       tableConfig: [
         { prop: "activityName", label: "活动名称"},
        { prop: "address", label: "活动地点" },
        { prop: "time", label: "活动时间" },
        { prop: "name", label: "发起人" },
      ],
      tableList: [
        {
          activityName:'ffff'
        }
      ],
    }
  },
  methods:{
    paginationChange(item){

    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
