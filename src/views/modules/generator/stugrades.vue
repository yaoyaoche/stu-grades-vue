<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-row class="padding-10-0">
          <el-button @click="getDataList()">查询</el-button>
          <el-button type="primary" @click="rank()">排名</el-button>
          <el-upload
            :beforeUpload="beforeUpload"
            :showUploadList="false"
            :multiple="true"
            :show-file-list="false"
            class="upload-demo inline-block margin-right-10">
            <el-button type="primary" icon="el-icon-upload"> 导入文件 </el-button>
          </el-upload>
          <el-button type="primary" icon="el-icon-download"  @click="exportExcel">导出excel</el-button>
          <el-button v-if="isAuth('generator:stugrades:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
          <el-button v-if="isAuth('generator:stugrades:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
        </el-row>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        type="index"
        header-align="center"
        align="center"
        label="序号">
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
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <Rank v-if="rankVisible" ref="Rank"></Rank>
  </div>
</template>

<style>
  .inline-block {
    display: inline-block;
  }
</style>

<script>
  import AddOrUpdate from './stugrades-add-or-update'
  import Rank from './stugrades-rank.vue'
  export default {
    data () {
      return {
        dataForm: {
          key: '',
          id: '',
          chinese: '',
          maths: '',
          english: '',
          sum: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        rankVisible: false
      }
    },
    components: {
      AddOrUpdate,
      Rank
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/stugrades/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 排名
      rank () {
        this.rankVisible = true
        this.$nextTick(() => {
          this.$refs.Rank.getrank()
        })
      },
      // 导入excel
      beforeUpload (file) {
        // let _this = this
        console.log(file)
        let formData = new FormData()
        formData.append('file', file)
        // let config = {
        //   headers: {
        //     'Content-Type': 'multipart/form-data'
        //   }
        // }
        // 这里填写调用的后端Excel数据处理接口
        this.$http({
          url: this.$http.adornUrl('/generator/stugrades/excelin'),
          method: 'post',
          data: formData
        })
        this.$forceUpdate()
      },
      // 导出excel
      exportExcel () {
        this.$http({
          url: this.$http.adornUrl('/generator/stugrades/excelout'),
          method: 'post',
          data: this.clueList,
          responseType: 'blob'
        }).then(res => {
          const link = document.createElement('A')
          let blob = new Blob([res.data], { type: 'multipary/form-data' })
          link.style.display = 'none'
          link.href = URL.createObjectURL(blob)
          link.setAttribute('download', decodeURI('学生成绩表.xlsx'))
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link)
          console.log(res)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/generator/stugrades/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
