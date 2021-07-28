<template>
  <el-dialog
    :title="'成绩排名'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="index"
        header-align="center"
        align="center"
        label="名次">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="姓名">
      </el-table-column>
      <el-table-column
        prop="chinese"
        header-align="center"
        align="center"
        label="语文">
      </el-table-column>
      <el-table-column
        prop="maths"
        header-align="center"
        align="center"
        label="数学">
      </el-table-column>
      <el-table-column
        prop="english"
        header-align="center"
        align="center"
        label="英语">
      </el-table-column>
      <el-table-column
        prop="sum"
        header-align="center"
        align="center"
        label="总分">
      </el-table-column>
    </el-table>
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
          sum: ''
        }
      }
    },
    methods: {
      getrank () {
        this.visible = true
        // this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/stugrades/rank'),
          method: 'get',
          params: this.$http.adornParams({
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.list
          } else {
            this.dataList = []
          }
          // this.dataListLoading = false
          this.$forceUpdate()
        })
      }
    }
  }
</script>
