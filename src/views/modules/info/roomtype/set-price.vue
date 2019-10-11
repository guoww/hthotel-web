<template>
  <el-dialog
    :title="'价格设置'"
    :close-on-click-modal="false"
    width="40%"
    :visible.sync="visible">
    <el-table
      :data="priceList"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="tags"
        header-align="center"
        align="center"
        label="会员级别">
        <template slot-scope="scope">
          {{scope.row.memberLevel.lvName}}
        </template>

      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="价格">
        <template slot-scope="scope">
          <el-input v-model="priceList[scope.$index].price"></el-input>
        </template>
      </el-table-column>
    </el-table>
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
        dataListSelections: [],
        dataList: [],
        dataListLoading: false,
        priceList: [],
        tempKey: -666666 // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
      }
    },
    created() {
      //this.getDataList()
    },
    methods: {
      init (id) {
        this.visible = true
        if(id>0) {
          this.$http({
            url: this.$http.adornUrl('/info/rmt/getPriceInfo?id='+id),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            console.log(data)
            this.priceList = data.priceList
          }).then(() => {

          })
        }else {

        }

      },
      // 表单提交
      dataFormSubmit () {
        console.log(this.priceList)
        this.$http({
          url: this.$http.adornUrl(`/info/rmt/setPrice`),
          method: 'post',
          data: JSON.stringify(this.priceList)
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
    }
  }
</script>

<style>
  .cp-unit{
    margin-left: 10px;
  }
</style>
