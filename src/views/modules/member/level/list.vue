<template>
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-button v-if="isAuth('sys:role:save')" type="primary" @click="addOrUpdateHandle(0)">新增</el-button>
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
        prop="id"
        header-align="center"
        align="center"
        width="80"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="lvName"
        header-align="center"
        align="center"
        label="等级名称">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="会员权益">
        <template slot-scope="scope">
          <el-tag type="warn" v-if="scope.row.registCounpon>0" class="mrg-bt-10">{{scope.row.registCounpon}}优惠券{{scope.row.registCpNum}}张</el-tag>

          <el-tag type="warn" v-if="scope.row.tenCpNum>0" class="mrg-bt-10">10元优惠券{{scope.row.tenCpNum}}张</el-tag>

          <el-tag type="warn" v-if="scope.row.twentyCpNum>0" class="mrg-bt-10">20元优惠券{{scope.row.twentyCpNum}}张</el-tag>

          <el-tag type="warn" v-if="scope.row.thirtyCpNum>0" class="mrg-bt-10">30元优惠券{{scope.row.thirtyCpNum}}张</el-tag>

          <el-tag type="warn" v-if="scope.row.fiftyCpNum>0" class="mrg-bt-10">50元优惠券{{scope.row.fiftyCpNum}}张</el-tag>

          <el-tag type="warn" v-if="scope.row.outTime!=''" class="mrg-bt-10">离店时间:{{scope.row.outTime}}</el-tag>

          <el-tag type="warn" v-if="scope.row.bookSave!=''" class="mrg-bt-10">预定保留:{{scope.row.bookSave}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        width="180"
        label="创建时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:role:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
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

    <level-edit v-if="levelEditVisible" ref="levelEdit" @refreshDataList="getDataList"></level-edit>
  </div>
</template>

<script>
  import levelEdit from './level-edit'

  export default {
    data () {
      return {
        dataForm: {
          roleName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        levelEditVisible: false
      }
    },
    components: {
      levelEdit
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/mb/level/getPage'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize
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
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.roleId
        })
      },
      addOrUpdateHandle (id) {
        this.levelEditVisible = true
        this.$nextTick(() => {
          this.$refs.levelEdit.init(id)
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
<style>
  .mrg-bt-10{
    margin-bottom: 10px;
  }
</style>
