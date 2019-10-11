<template>
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.roleName" placeholder="角色名称" clearable></el-input>
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
        prop="id"
        header-align="center"
        align="center"
        width="80"
        label="ID">
        <template slot-scope="scope">
          {{scope.$index+1}}
        </template>
      </el-table-column>
      <el-table-column
        prop="img"
        header-align="center"
        align="center"
        label="展示图">
        <template slot-scope="scope">
          <img :src="scope.row.img" width="100px">
        </template>
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="套餐名称">
        <template slot-scope="scope">
          {{scope.row.name}}
        </template>

      </el-table-column>
      <el-table-column
        prop="price"
        header-align="center"
        align="center"
        label="价格">
        <template slot-scope="scope">
          <el-tag>{{scope.row.price}}</el-tag>
          元
        </template>
      </el-table-column>
      <el-table-column
        prop="manNum"
        header-align="center"
        align="center"
        label="可叠加优惠券">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.useCounpon==0">是</el-tag>
          <el-tag v-else type="info">否</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="isEnabled"
        header-align="center"
        align="center"
        label="是否启用">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.isEnabled==0">是</el-tag>
          <el-tag v-else type="info">否</el-tag>
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

  </div>
</template>

<script>

  export default {
    data () {
      return {
        tagsList: ['WIFI','有窗户','空调','冰箱','一次性用品'],
        manNumList:[
          {'label':'1人','value': 1},{'label':'2人','value': 2},{'label':'3人','value': 3},
          {'label':'4人','value': 4},{'label':'5人','value': 5},{'label':'6人','value': 6}
        ],
        bedList: [
          {label:'1.5米床*1',value: 1},
          {label:'1.5米床*2',value: 2},
          {label:'1.8米床*1',value: 3},
          {label:'1.8米床*2',value: 4},
          {label:'2米大床*1',value: 5},
          {label:'2米大床*2',value: 6}
        ],
        statusList: [{label: '有房',value: 0},{label: '满房', value: 1}],
        dataForm: {
          roleName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        setPriceVisible: false
      }
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/mkt/dispkg/getPage'),
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
        this.$router.push({
          path: '/mkt/dispkg/edit',
          query: {'id': id}
        })
      },
      setPriceHandler (id) {
        this.setPriceVisible = true
        this.$nextTick(()=>{
          this.$refs.setPrice.init(id)
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
