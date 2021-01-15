<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="姓名" prop="stuName">
      <el-input v-model="dataForm.stuName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="年龄" prop="stuAge">
      <el-input v-model="dataForm.stuAge" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="籍贯" prop="stuJg">
      <el-input v-model="dataForm.stuJg" placeholder=""></el-input>
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
    data () {
      return {
        visible: false,
        dataForm: {
          stuId: 0,
          stuName: '',
          stuAge: '',
          stuJg: ''
        },
        dataRule: {
          stuName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          stuAge: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ]
          // stuJg: [
          //   { required: true, message: '不能为空', trigger: 'blur' }
          // ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.stuId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.stuId) {
            this.$http({
              url: this.$http.adornUrl(`/generator/stu/info/${this.dataForm.stuId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.stuName = data.stu.stuName
                this.dataForm.stuAge = data.stu.stuAge
                this.dataForm.stuJg = data.stu.stuJg
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/stu/${!this.dataForm.stuId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'stuId': this.dataForm.stuId || undefined,
                'stuName': this.dataForm.stuName,
                'stuAge': this.dataForm.stuAge,
                'stuJg': this.dataForm.stuJg
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
