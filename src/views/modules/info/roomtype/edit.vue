<template>
  <div>
    <el-form ref="form" :inline="true" :model="form" :rules="formRules" label-width="80px">
      <el-form-item label="房型图" prop="images" style="width:100%;">
        <div class="rmt_img_list">
          <div v-for="item in form.images" class="rmt_img_item">
            <img :src="baseImgPath+item">
          </div>

          <div>
            <el-upload
              class="avatar-uploader"
              action="http://localhost:8002/htsys/file/upload"
              :show-file-list="false"
              style="margin-top:10px"
              :on-success="handleAvatarSuccess"
              :before-upload="beforeAvatarUpload">
              <el-button type="primary">上传房型环境图</el-button>
            </el-upload>
          </div>

        </div>
      </el-form-item>
      <el-form-item label="房型名称" prop="name" style="width: 48%;">
        <el-input v-model="form.name" style="width:190px"></el-input>
      </el-form-item>
      <el-form-item label="床型" prop="bed" style="width: 48%;">
        <el-select v-model="form.bed" placeholder="请选择床型">
          <el-option v-for="item in bedList" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="可住人数" prop="manNum" style="width: 48%;">
        <el-select v-model="form.manNum" placeholder="请选择可住人数">
          <el-option v-for="item in manNumList" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="状态" prop="status" style="width: 48%;">
        <el-select v-model="form.status" placeholder="房间状态">
          <el-option v-for="item in statusList" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="是否启用" prop="isEnabled" style="width: 48%;">
        <el-switch v-model="form.isEnabled"></el-switch>
      </el-form-item>
      <el-form-item label="标签" prop="tags" style="width: 48%;">
        <el-checkbox-group v-model="form.tags">
          <el-checkbox-button v-for="tag in tagsList" :label="tag" :key="tag">{{tag}}</el-checkbox-button>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="市场价" prop="updateTime" style="width: 48%;">
        <el-input v-model="form.price" type="text"/>
      </el-form-item>
      <el-form-item label="创建时间" prop="createTime" style="width: 48%;">
        <el-input v-model="form.createTime" type="text" readonly="readonly"/>
      </el-form-item>
      <el-form-item style="width:100%;text-align: center;">
        <el-button type="primary" @click="onSubmit">立即提交</el-button>
        <el-button @click="goBack">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<style>
  .rmt_img_item{
    display:inline-block;
    width: 200px;
    height: 200px;
    border: 1px dashed #e2e3e3;
    padding: 5px;
    border-radius: 5px;
    margin-right: 10px;
  }
  .rmt_img_item>img{
    width:100%;
    height: 100%;
  }
</style>
<script>
export default {
  data() {
    return {
      baseImgPath: 'http://localhost:8002/htsys/img/upload/',
      tagsList: ['WIFI','有窗户','空调','冰箱','一次性用品'],
      manNumList:[
        {'label':'1人','value': 1},{'label':'2人','value': 2},{'label':'3人','value': 3},
        {'label':'4人','value': 4},{'label':'5人','value': 5},{'label':'6人','value': 6}
      ],
      bedList: [
        {label:'1.5米床*1',value: 1},
        {label:'1.5米床*2',value: 2},
        {label:'1.8米床*1',value: 3},
        {label:'1.8米床*2',value: 4},
        {label:'2米大床*1',value: 5},
        {label:'2米大床*2',value: 6}
      ],
      statusList: [{label: '有房',value: 0},{label: '满房', value: 1}],
      form: {
        id: 0,
        name: '',
        bed: '',
        manNum: 0,
        isEnabled: false,
        region: 78,
        createTime: '',
        updateTime: '',
        type: [],
        resource: '',
        desc: '',
        images: [],
        tags: [],
        price: 0
      },
      formRules: {
        name: [
          { required: true, message: '房型名称不能为空', trigger: 'blur' }
        ],
        bed: [
          { required: true, message: '请选择床型', trigger: 'blur' }
        ],
        manNum: [
          { required: true, message: '请选择可住人数', trigger: 'blur'}
        ],
        status: [
          { required: true, message: '请选择房间状态', trigger: 'blur'}
        ]
      }
    }
  },
  activated(){
    var createTime = this.form.createTime
    var thisTime = this.timeNow()
    if(!createTime) {
      this.form.createTime = thisTime
    }
    this.form.updateTime = thisTime

    var thisId = this.$route.query.id
    console.log(thisId)
    if(thisId) {
      this.getRoomType(thisId)
    }
  },
  methods: {
    getRoomType(id) {
      this.$http({
        url: this.$http.adornUrl('/info/rmt/getOne?id='+id),
        method: 'get'
      }).then(({data}) => {
        if(data&&data.code ===0) {
          console.log(data)
          var isEnabled = data.rmt.isEnabled
          isEnabled = isEnabled===1?true:false;
          data.rmt.isEnabled = isEnabled
          this.form=data.rmt
        }
      })
    },
    goBack () {
      this.$router.push({
        path: '/info-roomtype/list'
      })
    },
    timeNow () {
      return this.$moment().utc().format('YYYY-MM-DD HH:mm:ss') + ' ' + this.$moment().utc().format('dddd')
    },
    handleAvatarSuccess(res, file) {
      console.log(res)
      var url = "http://localhost:8002/htsys/img/upload/"+res.filename;
      this.form.images.push(res.filename)
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
    onSubmit() {
      console.log(this.timeNow())
      console.log(this.form)
      this.$refs['form'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl('/info/rmt/save'),
            method: 'post',
            data: this.$http.adornData({
              'id': this.form.id || undefined,
              'name': this.form.name,
              'manNum': this.form.manNum || 0,
              'images': this.form.images || [],
              'status': this.form.status,
              'tags' : this.form.tags,
              'isEnabled': this.form.isEnabled==true ? 1 : 0,
              'bed': this.form.bed,
              'updateTime': this.form.updateTime,
              'createTime': this.form.createTime ,
              'price': this.form.price
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.$router.push({
                    path: '/info-roomtype/list'
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
