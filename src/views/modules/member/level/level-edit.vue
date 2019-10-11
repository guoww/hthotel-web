<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :inline="true" :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="等级名称" prop="name" label-width="100px">
        <el-input v-model="dataForm.lvName" placeholder="等级名称" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="积分率" prop="name" label-width="100px">
        <el-input v-model="dataForm.integralRate" placeholder="消费一块获得的积分数" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="注册优惠券" prop="remark" label-width="100px">
        <el-radio-group v-model="dataForm.registCounpon" style="width: 280px;">
          <el-radio v-for="rgtCp in registCpList" :label="rgtCp.label" style="margin-left: 30px">{{rgtCp.txt}}</el-radio>
        </el-radio-group>
        <el-input-number v-model="dataForm.registCpNum" :min="1" :max="10" style="width: 120px;margin-left:30px;"></el-input-number>
        <span class="cp-unit">张</span>
      </el-form-item>
      <el-form-item size="mini" label="10元优惠券" label-width="100px">
        <el-input-number v-model="dataForm.tenCpNum" :min="0" :max="10" style="width: 120px;margin-left:30px;"></el-input-number>
        <span class="cp-unit">张</span>
      </el-form-item>
      <el-form-item size="mini" label="20元优惠券" label-width="100px">
        <el-input-number v-model="dataForm.twentyCpNum" :min="0" :max="10" style="width: 120px;margin-left:20px;"></el-input-number>
        <span class="cp-unit">张</span>
      </el-form-item>
      <el-form-item size="mini" label="30元优惠券" label-width="100px">
        <el-input-number v-model="dataForm.thirtyCpNum" :min="0" :max="10" style="width: 120px;margin-left:30px;"></el-input-number>
        <span class="cp-unit">张</span>
      </el-form-item>
      <el-form-item size="mini" label="50元优惠券" label-width="100px">
        <el-input-number v-model="dataForm.fiftyCpNum" :min="0" :max="10" style="width: 120px;margin-left:20px;"></el-input-number>
        <span class="cp-unit">张</span>
      </el-form-item>
      <el-form-item size="mini" label="退房时间" label-width="100px">
        <el-radio-group v-model="dataForm.outTime">
          <el-radio v-for="ot in outTimeList" :label="ot.label" style="margin-left: 30px">{{ot.label}}</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item size="mini" label="预定保留" label-width="100px">
        <el-radio-group v-model="dataForm.bookSave">
          <el-radio v-for="bt in bookTimeList" :label="bt.label" style="margin-left: 30px">{{bt.label}}</el-radio>
        </el-radio-group>
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
        registCpList:[
          {txt:'30元',label:30},
          {txt:'50元',label:50},
          {txt:'70元',label:70},
        ],
        outTimeList:[
          {label:'12:00'},
          {label:'13:00'},
          {label:'14:00'}
        ],
        bookTimeList:[
          {label:'18:00'},
          {label:'19:00'},
          {label:'20:00'}
        ],
        dataForm: {
          id: 0,
          lvName: '',
          remark: '',
          integralRate:1,
          registCounpon: '',
          registCpNum: 0,
          tenCpNum: 0,
          twentyCpNum: 0,
          thirtyCpNum: 0,
          fiftyCpNum: 0,
          outTime: '',
          bookSave: ''
        },
        dataRule: {
          lvName: [
            { required: true, message: '等级名称不能为空', trigger: 'blur' }
          ]
        },
        tempKey: -666666 // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        if(id>0) {
          this.$http({
            url: this.$http.adornUrl('/mb/level/getOne?id='+id),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            //this.menuList = treeDataTranslate(data, 'menuId')
            console.log(data)
            this.dataForm = data.memberLevel
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
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            console.log(this.dataForm)
            this.$http({
              url: this.$http.adornUrl(`/mb/level/save`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'registCounpon': this.dataForm.registCounpon,
                'registCpNum': this.dataForm.registCpNum,
                'tenCpNum': this.dataForm.tenCpNum,
                'twentyCpNum': this.dataForm.twentyCpNum,
                'thirtyCpNum': this.dataForm.thirtyCpNum,
                'fiftyCpNum': this.dataForm.fiftyCpNum,
                'outTime': this.dataForm.outTime,
                'bookSave':this.dataForm.bookSave,
                'integralRate':this.dataForm.integralRate
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
