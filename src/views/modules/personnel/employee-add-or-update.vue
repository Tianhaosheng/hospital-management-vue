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
      label-width="80px"
    >
      <el-form-item label="员工姓名" prop="employeeName">
        <el-input
          v-model="dataForm.employeeName"
          placeholder="员工姓名"
        ></el-input>
      </el-form-item>
      <el-form-item label="员工性别" prop="employeeSex">
        <el-input
          v-model="dataForm.employeeSex"
          placeholder="员工性别"
        ></el-input>
      </el-form-item>
      <el-form-item label="上级领导" prop="employeeSuperiorId">
        <el-input
          v-model="dataForm.employeeSuperiorId"
          placeholder="上级领导"
        ></el-input>
      </el-form-item>
      <el-form-item label="员工职位" prop="employeePositionId">
        <!-- <el-input
          v-model="dataForm.employeePositionId"
          placeholder="员工职位"
        ></el-input> -->
        <el-select v-model="value" placeholder="请选择">
          <el-option
            v-for="item in position_options"
            :key="item.id"
            :label="item.positionName"
            :value="item.id"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="员工描述" prop="employeeDescription">
        <el-input
          v-model="dataForm.employeeDescription"
          placeholder="员工描述"
        ></el-input>
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
      position_options: [
        {
          value: "选项1",
          label: "黄金糕",
        },
      ],
      value: "",
      visible: false,
      dataForm: {
        id: 0,
        employeeName: "",
        employeeSex: "",
        employeeSuperiorId: "",
        employeePositionId: "",
        employeeDescription: "",
      },
      dataRule: {
        employeeName: [
          { required: true, message: "员工姓名不能为空", trigger: "blur" },
        ],
        employeeSex: [
          { required: true, message: "员工性别不能为空", trigger: "blur" },
        ],
        // employeeSuperiorId: [
        //   { required: true, message: '上级领导不能为空', trigger: 'blur' }
        // ],
        value: [
          { required: true, message: "员工职位不能为空", trigger: "blur" },
        ],
        // employeeDescription: [
        //   { required: true, message: '员工描述不能为空', trigger: 'blur' }
        // ]
      },
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;

      this.$http({
        url: this.$http.adornUrl("/personnel/employee/position/list"),
        method: "get",
      }).then(({ data }) => {
        this.position_options = data.data;
      });

      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/personnel/employee/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.employeeName = data.employee.employeeName;
              this.dataForm.employeeSex = data.employee.employeeSex;
              this.dataForm.employeeSuperiorId = data.employee.employeeSuperiorId;
              this.value = data.employee.employeePositionId;
              this.dataForm.employeeDescription = data.employee.employeeDescription;
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
              `/personnel/employee/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              employeeName: this.dataForm.employeeName,
              employeeSex: this.dataForm.employeeSex,
              employeeSuperiorId: this.dataForm.employeeSuperiorId,
              employeePositionId: this.value,
              employeeDescription: this.dataForm.employeeDescription,
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
