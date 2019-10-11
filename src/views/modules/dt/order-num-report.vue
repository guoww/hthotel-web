<template>
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-date-picker
          v-model="startTime"
          type="date"
          placeholder="开始日期">
        </el-date-picker>
        至
        <el-date-picker
          v-model="endTime"
          type="date"
          placeholder="结束日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      :summary-method="getSummaries"
      border
      show-summary
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        header-align="center"
        align="center"
        width="80"
        label="序号">
        <template slot-scope="scope">
          {{scope.$index+1}}
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="日期">
        <template slot-scope="scope">
          {{scope.row.dayStr}}
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="待支付">
        <template slot-scope="scope">
          <el-tag type="success">
            {{scope.row.waitPay}}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="已支付"
      width="120px">
        <template slot-scope="scope">
          <el-tag type="danger">
            {{scope.row.paied}}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="已取消">
        <template slot-scope="scope">
          <el-tag type="info">{{scope.row.cancle}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="已完成">
        <template slot-scope="scope">
          <el-tag type="warn">{{scope.row.finished}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="全部订单">
        <template slot-scope="scope">
          {{scope.row.allOrder}}
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
        dataForm: {
          roleName: ''
        },
        startTime: null,
        endTime: null,
        keyword: null,
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        levelEditVisible: false
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
          url: this.$http.adornUrl('/dt/report/orderNumReport'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'startTime': this.startTime,
            'endTime': this.endTime
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
      },
      getSummaries(params){
        console.log("getSummaries:")
        console.log(params)
        var smWp=0
        var smPed =0
        var smCl = 0
        var smAll = 0
        var smFd = 0
        var datas = params.data
        for(var i=0;i<datas.length;i++) {
          smWp+= datas[i].waitPay
          smPed += datas[i].paied
          smCl += datas[i].cancle
          smFd += datas[i].finished
          smAll += datas[i].allOrder
        }

        return ['合计','',smWp,smPed,smCl,smFd,smAll]
      }
    }
  }
</script>
<style>
  .mrg-bt-10{
    margin-bottom: 10px;
  }
</style>
