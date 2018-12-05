<template>
  <div>
    <br>
    <el-form :inline="true">
      <el-form-item label="活动名称">
        <el-input v-model="searchMap.name" placeholder="活动名称"></el-input>
      </el-form-item>
      <el-form-item label="活动日期">
        <el-date-picker v-model="searchMap.starttime_1" type="date" placeholder="选择开始日期"></el-date-picker>
        <el-date-picker v-model="searchMap.starttime_2" type="date" placeholder="选择结束日期"></el-date-picker>
      </el-form-item>
      <el-button @click="fetchData()" type="primary">查询</el-button>
      <el-button @click="handleEdit('')" type="primary">新增</el-button>
    </el-form>
    <el-table :data="list" border style>
      <el-table-column prop="id" label="活动ID"></el-table-column>
      <el-table-column prop="name" label="活动名称"></el-table-column>
      <el-table-column prop="sponsor" label="主办方"></el-table-column>
      <el-table-column prop="starttime" label="开始日期"></el-table-column>
      <el-table-column prop="endtime" label="结束日期"></el-table-column>
      <el-table-column fixed="right" label="操作" width="100">
        <template slot-scope="scope">
          <el-button @click="handleEdit(scope.row.id)" type="text" size="small">修改</el-button>
          <el-button @click="handleDelete(scope.row.id)" type="text" size="small">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="fetchData"
      @current-change="fetchData"
      :current-page="currentPage"
      :page-sizes="[10,20,50,100]"
      :page-size="10"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
    <el-dialog title="编辑" :visible.sync="dialogFormVisible">
      <el-form label-width="80px">
        <el-form-item label="活动名称">
          <el-input v-model="pojo.name" placeholder="活动名称"></el-input>
        </el-form-item>
        <el-form-item label="基本地址">
          <el-input v-model="pojo.address" placeholder="基本地址"></el-input>
        </el-form-item>
        <el-form-item label="开始日期">
          <el-date-picker v-model="pojo.starttime" placeholder="开始日期"></el-date-picker>
        </el-form-item>
        <el-form-item label="截止日期">
          <el-date-picker v-model="pojo.endtime" placeholder="截止日期"></el-date-picker>
        </el-form-item>
        <el-form-item label="报名截止">
          <el-date-picker v-model="pojo.enrolltime" placeholder="报名截止"></el-date-picker>
        </el-form-item>
        <el-form-item label="活动详情">
          <el-input v-model="pojo.detail" placeholder="活动详情" type="textarea"></el-input>
        </el-form-item>
        <el-form-item label="选择城市">
          <el-select v-model="pojo.city" placeholder="请选择">
            <el-option v-for="item in cityList" :key="item.id" :label="item.name" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="是否可见">
          <el-switch active-value="1" inactive-value="0" v-model="pojo.status"></el-switch>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleSave()">保存</el-button>
          <el-button @click="dialogFormVisible=false">关闭</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
import gatheringApi from "@/api/gathering";
import cityApi from "@/api/city";
export default {
  data() {
    return {
      list: [],
      total: 0, //总记录数
      currentPage: 1, //当前页
      pageSize: 10, //每页大小
      searchMap: {}, //查询条件
      pojo: {},
      dialogFormVisible: false, //dialog是否显示
      cityList: [], //
      id: "" //当前修改的id
    };
  },
  created() {
    this.fetchData();
    cityApi.getList().then(response => {
      this.cityList = response.data;
    });
  },
  methods: {
    fetchData() {
      gatheringApi
        .search(this.currentPage, this.pageSize, this.searchMap)
        .then(response => {
          //   console.log(JSON.stringify(response));
          this.list = response.data.rows;
          this.total = response.data.total;
        });
    },
    handleSave() {
      gatheringApi.update(this.id, this.pojo).then(response => {
        this.$message({
          message: response.message,
          type: response.flag ? "success" : "error"
        });
        this.fetchData();
      });
      this.dialogFormVisible = false;
    },
    handleEdit(id) {
      this.id = id;
      this.dialogFormVisible = true;
      if (id == undefined) {
        this.pojo = {};
      } else {
        gatheringApi.findById(id).then(response => {
          if (response.flag) {
            this.pojo = response.data;
          }
        });
      }
    },
    handleDelete(id) {
      this.$confirm("确定要删除此记录吗？", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          gatheringApi.deleteById(id).then(response => {
            this.$message({
              message: response.message,
              type: response.flag ? "success" : "error"
            });
            if (response.flag) {
              this.fetchData();
            }
          });
        })
        .catch(() => {});
    }
  }
};
</script>

