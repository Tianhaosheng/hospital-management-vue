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
        <!-- <el-input
          v-model="dataForm.employeeSex"
          placeholder="员工性别"
        ></el-input> -->
        <el-radio v-model="dataForm.employeeSex" label="男">男</el-radio>
        <el-radio v-model="dataForm.employeeSex" label="女">女</el-radio>
      </el-form-item>
      <el-form-item label="员工照片" prop="employeePhoto">
        <!-- <el-input v-model="dataForm.logo" placeholder="品牌logo地址"></el-input> -->
        <single-upload v-model="dataForm.employeePhoto"></single-upload>
      </el-form-item>
      <el-form-item label="员工职位" prop="employeePositionId">
        <!-- <el-input
          v-model="dataForm.employeePositionId"
          placeholder="员工职位"
        ></el-input> -->
        <el-select
          v-model="dataForm.employeePositionId"
          @change="getEmployeeByPositionId(dataForm.employeePositionId)"
          placeholder="请选择"
        >
          <el-option
            v-for="item in position_options"
            :key="item.id"
            :label="item.positionName"
            :value="item.id"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="上级领导" prop="employeeSuperiorId">
        <!-- <el-input
          v-model="dataForm.employeeSuperiorId"
          placeholder="上级领导"
        ></el-input> -->
        <el-select
          v-model="dataForm.employeeSuperiorId"
          :disabled="is"
          :placeholder="msg"
        >
          <el-option
            v-for="item in employeeSuperior_options"
            :key="item.id"
            :label="item.employeeName"
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
import SingleUpload from "@/components/upload/singleUpload";
export default {
  components: { SingleUpload },
  data() {
    return {
      employeeSuperior_options: [
        {
          id: 1,
          employeeName: "测试",
        },
      ],
      position_options: [
        {
          id: 1,
          positionName: "院长",
        },
      ],
      msg: "请选择",
      is: false,
      visible: false,
      dataForm: {
        id: 0,
        employeeName: "",
        employeeSex: "",
        employeeSuperiorId: "",
        employeePositionId: "",
        employeeDescription: "",
        employeePhoto: "",
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
        employeePositionId: [
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

      //获取职位列表
      this.$http({
        url: this.$http.adornUrl("/personnel/employee/position/list"),
        method: "get",
      }).then(({ data }) => {
        this.position_options = data.data;
      });
      //根据id获取上级职位员工列表
      if (this.dataForm.id) {
        this.$http({
          url: this.$http.adornUrl(
            `/personnel/employee/employeeSuperiorByEmployeeId/list/${this.dataForm.id}`
          ),
          method: "get",
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.employeeSuperior_options = data.data;
            this.msg = "请选择";
            this.is = false;
          } else {
            this.msg = data.msg;
            this.is = true;
          }
        });
      } else {
        this.msg = "请先选择员工职位";
        this.is = true;
      }

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
              this.dataForm.employeeSuperiorId =
                data.employee.employeeSuperiorId;
              this.dataForm.employeePositionId =
                data.employee.employeePositionId;
              this.dataForm.employeeDescription =
                data.employee.employeeDescription;
              this.dataForm.employeePhoto = data.employee.employeePhoto;
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
              employeePositionId: this.dataForm.employeePositionId,
              employeeDescription: this.dataForm.employeeDescription,
              employeePhoto: this.dataForm.employeePhoto,
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
    //查询某个职位的员工
    getEmployeeByPositionId(employeePositionId) {
      this.dataForm.employeeSuperiorId = "";
      this.$http({
        url: this.$http.adornUrl(
          `/personnel/employee/employeeSuperiorByPositionId/list/${this.dataForm.employeePositionId}`
        ),
        method: "get",
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.employeeSuperior_options = data.data;
          this.msg = "请选择";
          this.is = false;
        } else {
          this.msg = data.msg;
          this.is = true;
        }
      });
    },
  },
};
</script>
