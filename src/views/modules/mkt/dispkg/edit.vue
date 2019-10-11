<template>
  <el-form  :model="dataForm" :inline="true" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <div style="width: 100%;height:52px;padding: 20px;background-color: #f7f7f7;border-radius: 5px;margin-bottom: 10px;">
      <span style="margin-top: -11px;float: right;">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">提交</el-button>
    </span>
    </div>
    <el-form-item label="展示图" prop="img" style="width: 900px">
      <div>
        <img :src="dataForm.img" style="width: 200px;height: 200px">
      </div>
      <div style="display:inline-block;margin-top:20px;">
        <el-upload
          action="http://localhost:8002/htsys/file/upload"
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          :before-upload="beforeAvatarUpload">
          <el-button>展示图上传</el-button>
        </el-upload>
      </div>
    </el-form-item>
    <el-form-item label="套餐名称" label-width="100px">
      <el-input v-model="dataForm.name" prop="name" placeholder="活动名称" style="width: 220px"></el-input>
    </el-form-item>
    <el-form-item label="套餐价格" label-width="100px">
      <el-input v-model="dataForm.price" type="number" prop="qgName" placeholder="价格" style="width: 220px"></el-input>
    </el-form-item>
    <el-form-item label="可叠加优惠券"  label-width="100px" style="display: inline-block">
      <el-switch
        v-model="dataForm.useCounpon">
      </el-switch>
    </el-form-item>
    <el-form-item label="是否启用" label-width="100px" style="display: inline-block;margin-left: 20px">
      <el-switch
        v-model="dataForm.isEnabled">
      </el-switch>
    </el-form-item>
    <el-form-item label="套餐内容">
      <el-input type="textarea" prop="goods" v-model="dataForm.goods" placeholder="套餐所包含的商品或服务"></el-input>
    </el-form-item>
    <div>
      <script :id="ueId" class="ueditor-box" type="text/plain" style="width: 100%; height: 800px;">{{dataForm.intros}}</script>
    </div>
  </el-form>

</template>

<script>
import ueditor from 'ueditor'
  export default {
    data () {
      return {
        ue: null,
        ueId: `J_ueditorBox_${new Date().getTime()}`,
        ueContent: '',
        visible: false,
        menuList: [],
        menuListTreeProps: {
          label: 'name',
          children: 'children'
        },
        roomTypeList: [],
        imgUrl:'',
        dataForm: {
          id: 0,
          name: '',
          createTime: '',
          price: '',
          useCounpon: '',
          isEnabled: 0,
          intros:'',
          img:'http://pavo.elongstatic.com/i/mobile180/nw_000fGdS8.jpg',
        },
        dataRule: {
          dataForm:{
            name: [
              { required: true, message: '套餐名称不能为空', trigger: 'blur' }
            ],
            price: [
              { required: true, message: '套餐价格不能为空', trigger: 'blur' }
            ],
            goods: [
              { required: true, message: '套餐内容不能为空', trigger: 'blur' }
            ]
          }
        },
        tempKey: -666666 // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
      }
    },
    activated (){

    },
    mounted () {
      this.ue = ueditor.getEditor(this.ueId)
      let id = this.$route.query.id
      console.log(id)
      if(id&&id>0)this.init(id)

    },
    methods: {
      init (id) {
        console.log(id)
        this.dataForm.id = id || 0
        this.visible = true
        if(id>0) {
          this.$http({
            url: this.$http.adornUrl('/mkt/dispkg/getOne?id='+id),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            //this.menuList = treeDataTranslate(data, 'menuId')
            console.log(data)
            //var dispkg = JSON.parse(data.disPkg)
            var dispkg = data.disPkg
            console.log(dispkg.img)
            dispkg.isEnabled = dispkg.isEnabled==0?true:false
            dispkg.useCounpon = dispkg.useCounpon==0?true:false
            this.dataForm = dispkg

            this.dataForm.img = dispkg.img
            console.log(this.dataForm.img)
            this.ue.ready(() => {
              this.ue.setContent(dispkg.intros)
            })
          })
        }else {
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }

      },
      handleAvatarSuccess(res, file) {
        console.log(res)
        this.dataForm.img="http://localhost:8002/htsys/img/upload/"+res.filename
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
      // 表单提交
      dataFormSubmit () {
        this.ue.ready(() => {
          this.dataForm.intros = this.ue.getContent()
        })
        console.log(this.dataForm)
        if(!this.dataForm.img||this.dataForm.img==''){
          this.$message({
            message: '请上传套餐展示图',
            type: 'error',
            duration: 1500,
          })
          return false
        }
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            console.log(this.dataForm)
            this.$http({
              url: this.$http.adornUrl(`/mkt/dispkg/save`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'useCounpon': this.dataForm.useCounpon ? 0 : 1,
                'isEnabled': this.dataForm.isEnabled ? 0: 1,
                'intros': this.dataForm.intros,
                'price': this.dataForm.price,
                'img':this.dataForm.img,
                'goods':this.dataForm.goods
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.$router.push({
                      path: '/mkt-dispkg/list',
                    })
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
