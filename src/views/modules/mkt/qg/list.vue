<template>
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-button v-if="isAuth('sys:role:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
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
        prop="qgName"
        header-align="center"
        align="center"
        label="活动名称">
      </el-table-column>
      <el-table-column
        prop="phone"
        header-align="center"
        align="center"
        width="180"
        label="开始时间">
        <template slot-scope="scope">
          <el-tag style="margin-right: 5px;margin-bottom:5px;">{{scope.row.startTime}}</el-tag>
        </template>

      </el-table-column>
      <el-table-column
        prop="idCard"
        header-align="center"
        align="center"
        width="200px"
        label="结束时间">
        <template slot-scope="scope">
          <el-tag>{{scope.row.endTime}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="useCounpon"
        header-align="center"
        align="center"
        width="180"
        label="可叠加优惠券">
        <template scope="scope">
          <el-tag v-if="scope.row.useCounpon==1" type="danger">不可叠加使用</el-tag>
          <el-tag v-else type="success">可叠加使用</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="enable"
        header-align="center"
        align="center"
        width="180"
        label="是否启用">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.enable==1" type="danger">否</el-tag>
          <el-tag v-else type="success">是</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        width="180"
        label="状态">
        <template scope="scope">
          <el-tag v-if="scope.row.status==0" type="info">已过期</el-tag>
          <el-tag v-else type="success">进行中</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:role:delete')" type="text" size="small" @click="setRoomTypeHandler(scope.row.id)">设置房型</el-button>
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

    <qg-edit v-if="qgEditVisble" ref="qgEdit" @refreshDataList="getDataList"></qg-edit>

    <set-room-type v-if="setRoomTypeVisible" ref="setRoomType" @refreshDataList="getDataList"></set-room-type>
  </div>
</template>

<script>

  import qgEdit from './edit'
  import setRoomType from './set-roomtype'

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
        qgEditVisble: false,
        setRoomTypeVisible: false
      }
    },
    components:{
      qgEdit,setRoomType
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/mkt/qg/getPage'),
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
      queryOptionChange(){
        console.log(this.queryStatus)
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
        this.qgEditVisble = true
        this.$nextTick(()=>{
          this.$refs.qgEdit.init(id)
        })
      },
      setRoomTypeHandler (id) {
        this.setRoomTypeVisible = true
        this.$nextTick(()=>{
          this.$refs.setRoomType.init(id)
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
