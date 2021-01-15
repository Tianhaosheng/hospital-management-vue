<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="职位名" prop="positionName">
      <el-input v-model="dataForm.positionName" placeholder="职位名"></el-input>
    </el-form-item>
    <el-form-item label="上级" prop="positionSuperiorId">
      <el-input v-model="dataForm.positionSuperiorId" placeholder="上级"></el-input>
    </el-form-item>
    <el-form-item label="职位描述" prop="positionDescription">
      <el-input v-model="dataForm.positionDescription" placeholder="职位描述"></el-input>
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
          id: 0,
          positionName: '',
          positionSuperiorId: '',
          positionDescription: ''
        },
        dataRule: {
          positionName: [
            { required: true, message: '职位名不能为空', trigger: 'blur' }
          ],
          // positionSuperiorId: [
          //   { required: true, message: '上级不能为空', trigger: 'blur' }
          // ],
          // positionDescription: [
          //   { required: true, message: '职位描述不能为空', trigger: 'blur' }
          // ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/personnel/position/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.positionName = data.position.positionName
                this.dataForm.positionSuperiorId = data.position.positionSuperiorId
                this.dataForm.positionDescription = data.position.positionDescription
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
              url: this.$http.adornUrl(`/personnel/position/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'positionName': this.dataForm.positionName,
                'positionSuperiorId': this.dataForm.positionSuperiorId,
                'positionDescription': this.dataForm.positionDescription
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
