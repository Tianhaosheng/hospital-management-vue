<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="100px"
    >
      <el-form-item label="部门名" prop="departmentName">
        <el-input
          v-model="dataForm.departmentName"
          placeholder="部门名"
        ></el-input>
      </el-form-item>
      <el-form-item label="部门负责人" prop="departmentPrincipal">
        <!-- <el-input
          v-model="dataForm.departmentPrincipal"
          placeholder="部门负责人"
        ></el-input> -->
        <el-cascader
          :props="defaultParams"
          placeholder="试试搜索：张三"
          :options="options"
          filterable
          clearable
          v-model="dataForm.departmentPrincipal"
          :show-all-levels="false"
        ></el-cascader>
      </el-form-item>
      <el-form-item label="启用状态" prop="enable">
        <!-- <el-input v-model="dataForm.enable" placeholder="启用状态[0 - 禁用，1 - 启用]"></el-input> -->
        <el-switch
          v-model="dataForm.enable"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      options: [
        {
          id: 1,
          name: "院长",
          children: [
            {
              id: 1,
              name: "张三",
            },
          ],
        },
      ],
      defaultParams: {
        label: "name",
        value: "id",
        children: "children",
      },
      visible: false,
      dataForm: {
        id: 0,
        departmentName: "",
        departmentPrincipal: "",
        enable: 1,
      },
      dataRule: {
        departmentName: [
          { required: true, message: "部门名不能为空", trigger: "blur" },
        ],
        departmentPrincipal: [
          { required: true, message: "部门负责人不能为空", trigger: "blur" },
        ],
        enable: [
          {
            required: true,
            message: "启用状态[0 - 禁用，1 - 启用]不能为空",
            trigger: "blur",
          },
        ],
      },
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/dept/department/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.departmentName = data.department.departmentName;
              this.dataForm.departmentPrincipal =
                data.department.departmentPrincipalPath;
              this.dataForm.enable = data.department.enable;
              this.options = data.employeeCascader
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/dept/department/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              departmentName: this.dataForm.departmentName,
              // departmentPrincipal: this.dataForm.departmentPrincipal,
              departmentPrincipal: this.dataForm.departmentPrincipal[1],
              enable: this.dataForm.enable,
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
              console.log(this.dataForm);
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
};
</script>
