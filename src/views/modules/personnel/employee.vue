<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('personnel:employee:save')"
          type="primary"
          @click="addOrUpdateHandle()"
          >新增</el-button
        >
        <el-button
          v-if="isAuth('personnel:employee:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      >
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="员工编号"
      >
      </el-table-column>
      <el-table-column
        prop="employeeName"
        header-align="center"
        align="center"
        label="员工姓名"
      >
      </el-table-column>
      <el-table-column
        prop="employeeSex"
        header-align="center"
        align="center"
        label="员工性别"
      >
      </el-table-column>
      <el-table-column
        prop="employeeSuperiorId"
        header-align="center"
        align="center"
        label="上级领导"
      >
      </el-table-column>
      <el-table-column
        v-if="ok"
        prop="employeePositionId"
        header-align="center"
        align="center"
        label="员工职位"
      >
      </el-table-column>
      <el-table-column
        prop="employeePositionName"
        header-align="center"
        align="center"
        label="员工职位"
      >
        <template slot-scope="scope">
          <router-link :to="{ path: 'personnel-position', query: { id: scope.row.employeePositionId } }">
            <a>{{ scope.row.employeePositionName }}</a>
          </router-link>
        </template>
      </el-table-column>
      <el-table-column
        prop="employeeDescription"
        header-align="center"
        align="center"
        label="员工描述"
      >
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.id)"
            >修改</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.id)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    >
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList"
    ></add-or-update>
  </div>
</template>

<script>
import Vue from 'vue';
import AddOrUpdate from "./employee-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: "",
      },
      id: 0,
      ok: false,
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
    };
  },
  components: {
    AddOrUpdate,
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      //接收参数
      var id = this.$route.query.id;
        if(id === undefined){
          this.id = null;
          console.log(this.id)
        }else{
          this.id = id;
          console.log(this.id)
        }

      this.$http({
        url: this.$http.adornUrl("/personnel/employee/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          id: this.id
        }),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
          console.log(this.dataList);

          this.$http({
            url: this.$http.adornUrl("/personnel/employee/getPositionName"),
            method: "post",
            data: this.$http.adornData(this.dataList, false),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              for (var i = 0; i < this.dataList.length; i++) {
                Vue.set(this.dataList[i],"employeePositionName",data.positionNames[i]);
              }
              console.log(this.dataList);
            } else {
              this.$message.error(data.msg);
            }
          });
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map((item) => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/personnel/employee/delete"),
          method: "post",
          data: this.$http.adornData(ids, false),
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              },
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
  },
};
</script>
