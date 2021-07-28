<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="姓名" prop="name">
      <el-input v-model="dataForm.name" placeholder="姓名"></el-input>
    </el-form-item>
    <el-form-item label="语文" prop="chinese">
      <el-input v-model="dataForm.chinese" placeholder="语文"></el-input>
    </el-form-item>
    <el-form-item label="数学" prop="maths">
      <el-input v-model="dataForm.maths" placeholder="数学"></el-input>
    </el-form-item>
    <el-form-item label="英语" prop="english">
      <el-input v-model="dataForm.english" placeholder="英语"></el-input>
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
          name: '',
          chinese: '',
          maths: '',
          english: '',
          sum: ''
        },
        dataRule: {
          name: [
            { required: true, message: '姓名不能为空', trigger: 'blur' }
          ],
          chinese: [
            { required: true, message: '语文不能为空', trigger: 'blur' }
          ],
          maths: [
            { required: true, message: '数学不能为空', trigger: 'blur' }
          ],
          english: [
            { required: true, message: '英语不能为空', trigger: 'blur' }
          ]
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
              url: this.$http.adornUrl(`/generator/stugrades/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.stuGrades.name
                this.dataForm.chinese = data.stuGrades.chinese
                this.dataForm.maths = data.stuGrades.maths
                this.dataForm.english = data.stuGrades.english
                this.dataForm.sum = data.stuGrades.sum
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
              url: this.$http.adornUrl(`/generator/stugrades/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'chinese': this.dataForm.chinese,
                'maths': this.dataForm.maths,
                'english': this.dataForm.english,
                'sum': this.dataForm.sum
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
