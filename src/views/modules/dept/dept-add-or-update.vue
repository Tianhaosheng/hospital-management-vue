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
      <el-form-item label="科室名字" prop="deptName">
        <el-input v-model="dataForm.deptName" placeholder="科室名字"></el-input>
      </el-form-item>
      <el-form-item label="科室负责人" prop="deptPrincipal">
        <!-- <el-input
          v-model="dataForm.deptPrincipal"
          placeholder="科室负责人"
        ></el-input> -->
        <el-cascader
          :props="defaultParams"
          placeholder="试试搜索：张三"
          :options="options"
          filterable
          clearable
          v-model="dataForm.deptPrincipal"
          :show-all-levels="false"
        ></el-cascader>
      </el-form-item>
      <el-form-item label="科室地址" prop="deptAddress">
        <el-input
          v-model="dataForm.deptAddress"
          placeholder="科室地址"
        ></el-input>
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
      <el-form-item label="所属部门" prop="department">
        <!-- <el-input v-model="dataForm.department" placeholder="所属部门"></el-input> -->
        <template>
          <el-select v-model="value" placeholder="请选择">
            <el-option
              v-for="item in department_options"
              :key="item.id"
              :label="item.departmentName"
              :value="item.departmentName"
              :disabled="!item.enable"
            >
            </el-option>
          </el-select>
        </template>
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
      department_options: [
        {
          value: "1",
          label: "门诊部",
        },
      ],
      value: "",
      visible: false,
      dataForm: {
        id: 0,
        deptName: "",
        deptPrincipal: "",
        deptAddress: "",
        enable: 1,
        department: "",
      },
      dataRule: {
        deptName: [
          { required: true, message: "科室名字不能为空", trigger: "blur" },
        ],
        deptPrincipal: [
          { required: true, message: "科室负责人不能为空", trigger: "blur" },
        ],
        deptAddress: [
          { required: true, message: "科室地址不能为空", trigger: "blur" },
        ],
        enable: [
          {
            required: true,
            message: "启用状态不能为空",
            trigger: "blur",
          },
        ],
        value: [
          { required: true, message: "所属部门不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;

      this.$http({
        url: this.$http.adornUrl("/dept/dept/department/list"),
        method: "get",
      }).then(({ data }) => {
        
        this.department_options = data.data
      });

      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/dept/dept/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.deptName = data.dept.deptName;
              this.dataForm.deptPrincipal = data.dept.deptPrincipalPath;
              this.dataForm.deptAddress = data.dept.deptAddress;
              this.dataForm.enable = data.dept.enable;
              this.value = data.dept.department;
              this.options = data.employeeCascader;
              console.log(this.value)
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
              `/dept/dept/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              deptName: this.dataForm.deptName,
              deptPrincipal: this.dataForm.deptPrincipal[1],
              deptAddress: this.dataForm.deptAddress,
              enable: this.dataForm.enable,
              department: this.value,
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
