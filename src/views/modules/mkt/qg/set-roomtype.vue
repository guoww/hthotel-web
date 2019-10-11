<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    width="90%"
    :visible.sync="visible">
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="房型名称">
        <template slot-scope="scope">
          {{scope.row.roomType.name}}
        </template>
      </el-table-column>
      <el-table-column
        prop="tags"
        header-align="center"
        align="center"
        label="普通会员价">
        <template slot-scope="scope">
          498元
        </template>

      </el-table-column>
      <el-table-column
        prop="bed"
        header-align="center"
        align="center"
        label="银牌会员价">
        <template slot-scope="scope">
          468元
        </template>
      </el-table-column>
      <el-table-column
        prop="manNum"
        header-align="center"
        align="center"
        label="金牌会员价">
        <template slot-scope="scope">
          438元
        </template>
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="抢购价">
        <template slot-scope="scope">
          <el-input v-if="qgRmList[scope.$index].isUsed!==true" v-model="qgRmList[scope.$index].qiangGouRm.qgPrice"></el-input>
          <span v-else style="color:red">该房型已被其他优惠活动占用</span>
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
        dataForm: {
          id: 0,
          qgName: '',
          startTime: '',
          endTime: '',
          useCounpon: '',
          enable: 0
        },
        qgRmList: [],
        dataRule: {
          dataForm:{
            qgName: [
              { required: true, message: '活动名称不能为空', trigger: 'blur' }
            ],
            startTime: [
              { required: true, message: '请选择开始时间', trigger: 'blur' }
            ],
            endTime: [
              { required: true, message: '请选择结束时间', trigger: 'blur' }
            ]
          }
        },
        tempKey: -666666 // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
      }
    },
    created() {
      //this.getDataList()
    },
    methods: {
      init (id) {
        console.log(id)
        this.dataForm.id = id || 0
        this.visible = true
        this.getDataList(id)
        if(id>0) {
          this.$http({
            url: this.$http.adornUrl('/mkt/qg/getOne?id='+id),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            //this.menuList = treeDataTranslate(data, 'menuId')
            console.log(data)
            var qg = data.qg
            qg.enable = qg.enable==0?true:false
            qg.useCounpon = qg.useCounpon==0?true:false
            this.dataForm = data.qg
          }).then(() => {

          })
        }else {

        }

      },
      getDataList (qgId) {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/mkt/qgrm/getList'),
          method: 'get',
          params: this.$http.adornParams({
            'qgId': qgId,
            'page': this.pageIndex,
            'limit': this.pageSize
          })
        }).then(({data}) => {
          console.log(data)
          this.dataList = data.list
          this.qgRmList = data.list
          this.dataListLoading = false
        })

      },
      clearQgRmListStatus () {
        var qgRms = this.qgRmList
        for(var i=0;i<qgRms.length;i++) {
          qgRms[i].status = false
        }
      },
      selectionChangeHandle (val) {
        console.log(val)
        this.dataListSelections = val
        var dtlist = this.qgRmList

        this.clearQgRmListStatus()

        if(val.length>0){
          for(var i=0;i<val.length;i++) {
            for(var j=0;j<dtlist.length;j++) {
              if(val[i].id===dtlist[j].roomTypeId) {
                dtlist[j].status = true
              }
            }
          }
        }
        console.log(this.qgRmList)
      },
      // 表单提交
      dataFormSubmit () {
        console.log(this.qgRmList)
        this.$http({
          url: this.$http.adornUrl(`/mkt/qgrm/save`),
          method: 'post',
          data: JSON.stringify(this.qgRmList)
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
