<template>
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.code" placeholder="优惠码" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('sys:role:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('sys:role:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
        width="80"
        label="序号">
        <template scope="scope">
          {{scope.$index+1}}
        </template>
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="名称">
      </el-table-column>
      <el-table-column
        prop="code"
        header-align="center"
        align="center"
        label="优惠码">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="类型">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.type==0" style="margin-right: 5px;margin-bottom:5px;">抵扣券</el-tag>
          <el-tag v-else type="warn" style="margin-right: 5px;margin-bottom:5px;">折扣券</el-tag>
        </template>
      </el-table-column>

      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="折扣">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.type==0" style="margin-right: 5px;margin-bottom:5px;">{{scope.row.discount}}元</el-tag>
          <el-tag v-else type="warn" style="margin-right: 5px;margin-bottom:5px;">{{scope.row.discount}}折</el-tag>
        </template>
      </el-table-column>

      <el-table-column
        prop="phone"
        header-align="center"
        align="center"
        width="180"
        label="有效期">
        <template slot-scope="scope">
          <el-tag style="margin-right: 5px;margin-bottom:5px;">{{scope.row.startTime}}</el-tag>
          <div style="width: 100%;">至</div>
          <el-tag style="margin-right: 5px;margin-bottom:5px;">{{scope.row.endTime}}</el-tag>
        </template>

      </el-table-column>

      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        width="180"
        label="来源">
        <template scope="scope">
          后台生成
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        width="180"
        label="状态">
        <template scope="scope">
          <el-tag v-if="scope.row.status==0" type="info">待分发</el-tag>
          <el-tag v-else-if="scope.row.status==1" type="warn">待使用</el-tag>
          <el-tag v-else-if="scope.row.status==2" type="success">已使用</el-tag>
          <el-tag v-else type="info">已过期</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:role:delete')" type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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

    <cpEdit v-if="cpEditVisble" ref="cpEdit" @refreshDataList="getDataList"></cpEdit>

  </div>
</template>

<script>

  import cpEdit from './edit'

  export default {
    data () {
      return {
        code:'',
        dataForm: {
          roleName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        cpEditVisble: false
      }
    },
    components:{
      cpEdit
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/mkt/counpon/getPage'),
          method: 'get',
          params: this.$http.adornParams({
            'pageNum': this.pageIndex,
            'limit': this.pageSize,
            'code': this.code
          })
        }).then(({data}) => {
          console.log(data)
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
        console.log(val)
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 删除
      deleteHandle (id) {
        this.$confirm('此操作将永久该记录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(()=>{
          this.$http({
            url: this.$http.adornUrl('/mkt/counpon/del'),
            method: 'get',
            params: this.$http.adornParams({
              'id': id
            })
          }).then(({data}) => {
            if(data.code==0) {
              this.$message({
                type: 'success',
                message: '删除成功!'
              });
            }
          })
        })
      },
      addOrUpdateHandle (id) {
        this.cpEditVisble = true
        this.$nextTick(()=>{
          this.$refs.cpEdit.init(id)
        })
      },
      getSelectObjName (selectList,val) {
        console.log(selectList)
        for(let index in selectList) {
          var it = selectList[index]
          if(it.value===val) return it.label
        }
      }
    }
  }
</script>
