<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :inline="true" :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="会员名称" prop="name" label-width="100px">
        <el-input v-model="dataForm.name" placeholder="会员名称" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="手机号码" prop="phone" label-width="100px">
        <el-input v-model="dataForm.phone" placeholder="手机号码" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="后备手机" prop="bkPhone" label-width="100px">
        <el-input v-model="dataForm.bkPhone" placeholder="后备手机" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="身份证号" prop="idCard" label-width="100px">
        <el-input v-model="dataForm.idCard" placeholder="身份证号" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="会员等级" prop="memberLevelId" label-width="100px">
        <el-input v-model="dataForm.memberLevel.lvName" placeholder="会员等级" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="邀请码" prop="code" label-width="100px">
        <el-input v-model="dataForm.code" placeholder="邀请码" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="积分" prop="integral" label-width="100px">
        <el-input v-model="dataForm.integral" placeholder="积分余额" style="margin-left: 30px;"></el-input>
      </el-form-item>
      <el-form-item label="余额" prop="gold" label-width="100px">
        <el-input v-model="dataForm.gold" placeholder="账号余额" style="margin-left: 30px;"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import memberView from './member-view'
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
    components:{
      memberView
    },
    methods: {
      init (id) {
        console.log(id)
        this.dataForm.id = id || 0
        this.visible = true
        if(id>0) {
          this.$http({
            url: this.$http.adornUrl('/mb/getOne?id='+id),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            console.log(data)
            this.dataForm = data.member
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
