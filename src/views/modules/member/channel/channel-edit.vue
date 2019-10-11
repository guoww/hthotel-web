<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="渠道名称" prop="name" label-width="100px">
        <el-input v-model="dataForm.name" placeholder="渠道名称"></el-input>
      </el-form-item>
      <el-form-item label="推广码" prop="code" label-width="100px">
        <el-input v-model="dataForm.code" readonly="readonly" placeholder="推广码"></el-input>
      </el-form-item>
      <el-form-item label="推广链接" prop="code" label-width="100px">
        <el-input v-model="dataForm.channelUrl" readonly="readonly" placeholder="推广链接"></el-input>
      </el-form-item>
      <el-form-item label="推广二维码" label-width="100px">
        <img :src="qrCode" style="width: 200px;height: 200px;">
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
        qrCode: '',
        dataForm: {
          id: 0,
          name: '',
          code: '',
          channelUrl: ''
        },
        dataRule: {
          name: [
            { required: true, message: '渠道名称不能为空', trigger: 'blur' }
          ]
        },
        tempKey: -666666 // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
      }
    },
    methods: {
      init (id) {
        console.log(id)
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
        })

        if(id>0) {
          this.$http({
            url: this.$http.adornUrl('/mb/channel/getOne?id='+id),
            method:'get',
            params: this.$http.adornParams()
          }).then((data)=>{
            console.log(data)
            this.dataForm = data.data.channel
            this.qrCode = 'http://localhost:8002/htsys/mb/channel/getQrcode?code='+this.dataForm.code
          })
        } else {
          this.$http({
            url: this.$http.adornUrl('/mb/channel/getCode'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            console.log(data)
            this.dataForm.code = data.code
            this.dataForm.channelUrl = "http:localhost:8002/tg/test?code="+data.code
            this.qrCode = 'http://localhost:8002/htsys/mb/channel/getQrcode?code='+data.code
          })
        }

      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            console.log(this.dataForm)
            this.$http({
              url: this.$http.adornUrl(`/mb/channel/save`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'code': this.dataForm.code,
                'channelUrl': this.dataForm.channelUrl
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
