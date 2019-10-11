<template>
  <div>
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="环境图" prop="userName">
        <div class="img-list">
          <div v-for="img in dataForm.envImages" style="display:inline-block;margin-right: 10px;">
            <img  :src="baseImgPath+img" class="avatar">
          </div>



        </div>
        <div style="display:inline-block">
          <el-upload
            class="avatar-uploader"
            action="http://localhost:8002/htsys/file/upload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
            <i class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </div>
      </el-form-item>
      <el-form-item label="酒店名称" prop="hotelName" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.hotelName" type="text" placeholder="酒店名称" style="width:300px"></el-input>
      </el-form-item>
      <el-form-item label="联系电话" prop="phone" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.phone" type="phone" placeholder="联系电话" style="width:300px"></el-input>
      </el-form-item>
      <el-form-item label="地址" prop="address">
        <el-input v-model="dataForm.address" placeholder="地址" style="width:500px"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
</div>

</template>
<style>
  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
  }
  .img-list{
    display: inline-block;
  }
  .img-list img{
    display: inline-block;
    border-radius: 10px;
    margin-right: 10px;
  }
</style>
<script>
  import { isMobile } from '@/utils/validate'
  export default {
    data () {
      var validateMobile = (rule, value, callback) => {
        if (!isMobile(value)) {
          callback(new Error('手机号格式错误'))
        } else {
          callback()
        }
      }
      return {
        visible: false,
        roleList: [],
        baseImgPath: 'http://localhost:8002/htsys/img/upload/',
        dataForm: {
          id: 0,
          hotelName: '',
          address: '',
          phone: '',
          envImages:[]
        },
        dataRule: {
          hotelName: [
            { required: true, message: '酒店名称不能为空', trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '酒店联系电话不能为空', trigger: 'blur' },
            { validator: validateMobile, trigger: 'blur' }
          ]
        }
      }
    },
    activated () {
      this.init()
    },
    methods: {
      handleAvatarSuccess(res, file) {
        console.log(res)
        var url = "http://localhost:8002/htsys/img/upload/"+res.filename;
        this.dataForm.envImages.push(url)
        this.imageUrl = URL.createObjectURL(file.raw);
      },
      beforeAvatarUpload(file) {
        var isJPG = file.type === 'image/jpeg';
        var isLt2M = file.size / 1024 / 1024 < 8;

        if (!isJPG) {
          this.$message.error('上传头像图片只能是 JPG 格式!');
        }
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 8MB!');
        }
        return isJPG && isLt2M;
      },
      init (id) {
        this.dataForm.id = id || 0
        console.log(123)
        this.$http({
          url: this.$http.adornUrl('/info/htsetting/getSetting'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({ data }) => {
          console.log(data.setting)
          var imgs = data.setting.envImages

          //data.setting.envImages = imgs
          console.log(data.setting)
          this.dataForm = data.setting
      })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl('/info/htsetting/save'),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'hotelName': this.dataForm.hotelName,
                'address': this.dataForm.address,
                'phone': this.dataForm.phone
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
