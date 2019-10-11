<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form  :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="活动名称" label-width="100px">
        <el-input v-model="dataForm.qgName" prop="qgName" placeholder="活动名称" style="width: 220px"></el-input>
      </el-form-item>
      <el-form-item label="开始时间" label-width="100px">
        <el-date-picker
          v-model="dataForm.startTime"
          type="datetime"
          value-format="yyyy-mm-dd HH:mm:ss"
          prop="startTime"
          placeholder="选择开始时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" label-width="100px">
        <el-date-picker
          v-model="dataForm.endTime"
          type="datetime"
          value-format="yyyy-mm-dd HH:mm:ss"
          prop="endTime"
          placeholder="选择结束时间">
        </el-date-picker>
      </el-form-item>
        <el-form-item label="可叠加优惠券"  label-width="100px" style="display: inline-block">
          <el-switch
            v-model="dataForm.useCounpon">
          </el-switch>
        </el-form-item>
        <el-form-item label="是否启用" label-width="100px" style="display: inline-block;margin-left: 20px">
          <el-switch
            v-model="dataForm.enable">
          </el-switch>
        </el-form-item>

    </el-form>
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
        menuList: [],
        menuListTreeProps: {
          label: 'name',
          children: 'children'
        },
        roomTypeList: [],
        dataForm: {
          id: 0,
          qgName: '',
          startTime: '',
          endTime: '',
          useCounpon: '',
          enable: 0,
          roomTypeList:[]
        },
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
    created (){

    },
    methods: {
      init (id) {
        console.log(id)
        this.dataForm.id = id || 0
        this.visible = true
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
            qg.roomTypeList = []
            this.dataForm = qg
          }).then(() => {
            this.$nextTick(() => {
              this.$refs['dataForm'].resetFields()
            })
          })
        }else {
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }

      },
      getRoomTypList() {
        this.$http({
          url: this.$http.adornUrl('/info/rmt/getFreeRm'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data})=>{
          console.log(data)
          this.roomTypeList = data.rms
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            console.log(this.dataForm)
            this.$http({
              url: this.$http.adornUrl(`/mkt/qg/save`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'qgName': this.dataForm.qgName,
                'useCounpon': this.dataForm.useCounpon ? 0 : 1,
                'enable': this.dataForm.enable ? 0: 1,
                'startTime': this.dataForm.startTime,
                'endTime': this.dataForm.endTime
              })
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
