<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form  :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="优惠券名称" label-width="100px">
        <el-input v-model="dataForm.name" prop="name" placeholder="名称" style="width: 220px"></el-input>
      </el-form-item>
      <el-form-item label="类型" label-width="100px">
        <el-select v-model="dataForm.type" prop="type" placeholder="请选择">
          <el-option
            v-for="item in typeOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="抵扣/折扣"  label-width="100px" style="display: inline-block">
        <el-input v-model="dataForm.discount" type="number" prop="discount" placeholder="请输入抵扣/折扣" style="width: 220px"></el-input>
      </el-form-item>
      <el-form-item label="有效期" label-width="100px">
        <el-date-picker
          v-model="dataForm.dateActive"
          type="daterange"
          align="right"
          unlink-panels
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          value-format="yyyy-MM-dd"
          :picker-options="pickerOptions"
          @change="activeDateChangeHandler">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="生成数量"  label-width="100px" style="display: inline-block">
        <el-input v-model="dataForm.num" type="number" prop="num" placeholder="本次生成的优惠券数量" style="width: 220px"></el-input>
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
        typeOptions:[
          {label:'抵扣券',value:0},
          {label:'折扣券',value:1}
        ],
        roomTypeList: [],
        dataForm: {
          id: 0,
          name: '',
          startTime: '',
          endTime: '',
          dateActive:'',
          type:0,
          discount:'',
          num:''
        },
        dataRule: {
          name: [
            { required: true, message: '优惠券名称不能为空', trigger: 'blur' }
          ],
          dateActive: [
            { required: true, message: '有效期', trigger: 'blur' }
          ],
          num: [
            { required: true, message: '请输入生成数量', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '请选择优惠类型', trigger: 'blur' }
          ],
          discount: [
            { required: true, message: '请输入抵扣/折扣', trigger: 'blur' }
          ]
        },
        tempKey: -666666, // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
        pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          }]
        }
      }
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
      activeDateChangeHandler (d) {
        this.dataForm.startTime = d[0]
        this.dataForm.endTime = d[1]
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/mkt/counpon/save`),
              method: 'post',
              data: this.$http.adornData({
                'name': this.dataForm.name,
                'type': this.dataForm.type,
                'num': this.dataForm.num,
                'startTime': this.dataForm.startTime,
                'endTime': this.dataForm.endTime,
                'discount':this.dataForm.discount
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
