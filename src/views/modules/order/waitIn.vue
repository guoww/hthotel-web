<template>
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="keyword" placeholder="会员名称/手机/身份证号" clearable style="width:200px;"></el-input>
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
        prop="orderNo"
        header-align="center"
        align="center"
        width="150"
        label="订单编号">
      </el-table-column>
      <el-table-column
        prop="orderType"
        header-align="center"
        align="center"
        label="订单类型">
        <template slot-scope="scope">
          {{getSelectObjName(orderTypeList,scope.row.orderType)}}
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="会员名称">
        <template slot-scope="scope">
          <span>
            {{scope.row.member.name}}
          </span>
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="联系电话">
        <template slot-scope="scope">
          <span>
            {{scope.row.member.phone}}
          </span>
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="身份证号">
        <template slot-scope="scope">
          <span>
            {{scope.row.member.idCard}}
          </span>
        </template>
      </el-table-column>
      <el-table-column
        prop="price"
        header-align="center"
        align="center"
        label="金额">
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="订单状态">
        <template slot-scope="scope">
          <el-tag>{{getSelectObjName(statusList,scope.row.status)}}</el-tag>
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
          <el-button v-if="isAuth('sys:role:update')" type="text" size="small" @click="orderDetailHandler(scope.row.id)">详情</el-button>
          <el-button v-if="isAuth('order:checkin')" type="text" size="small" @click="finishOrder(scope.row.id)">确认入住</el-button>
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

    <!-- 弹窗, 订单详情 -->
    <order-detail v-if="orderDetailVisible" ref="orderDetail"></order-detail>
  </div>
</template>

<script>
  import orderDetail from './order-detail'

  export default {
    data () {
      return {
        tagsList: ['WIFI','有窗户','空调','冰箱','一次性用品'],
        statusList: [
          {label: '待支付',value: 0},{label: '待入住', value: 1},
          {label: '待离店',value: 2},{label: '已完成',value:3},{label: '已取消',value:4}
        ],
        orderTypeList:[
          {label: '普通房型订单',value: 0},{label: '主题套餐订单', value: 1},
          {label: '抢购房型订单',value: 2},{label: '会员升级订单',value:3}
        ],
        keyword:'',
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        orderDetailVisible: false
      }
    },
    components: {
      orderDetail
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        var status = this.$route.query.status
        this.dataListLoading = true
        console.log(this.pageIndex+'--'+this.pageSize+'--'+status)
        this.$http({
          url: this.$http.adornUrl('/od/order/getPage'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'status': 1,
            'keyword':this.keyword
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
      finishOrder(orderId){
        var that = this
        this.$confirm('确定顾客已入住并完成该笔交易？','提示',{
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(()=>{
          console.log(orderId)
          this.$http({
            url: this.$http.adornUrl('/od/order/finishOrder'),
            method: 'get',
            params: this.$http.adornParams({
              'orderId': orderId,
            })
          }).then(()=>{
            this.$message({
              type: 'success',
              message: '订单已完成!',
              onClose:function(){
                that.getDataList()
              }
            })
          })

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
          path: '/info/order/edit',
          query: {'id': id}
        })
      },
      orderDetailHandler (id) {
        this.orderDetailVisible = true
        console.log(this.orderDetailVisible)
        this.$nextTick(() => {
          this.$refs.orderDetail.init(id)
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
