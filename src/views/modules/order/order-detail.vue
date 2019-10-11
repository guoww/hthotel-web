<template>
  <el-dialog
    title="订单详情"
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="80%"
    height="80%"
  >
    <el-form :inline="true" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">

      <div class="info-split">订单信息</div>
      <el-form-item label="订单编号" prop="type">
        <el-input v-model="order.orderNo" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="订单类型">
        <el-input v-model="order.orderTypeName" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="订单状态">
        <el-input class="menu-list__input" v-model="order.statusName" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="订单金额">
        <el-input v-model="order.price" placeholder="支付金额" style="width: 160px"></el-input>
        <span style="margin-left:5px;">元</span>
      </el-form-item>
      <el-form-item label="支付时间">
        <el-input v-model="order.payTime" placeholder="支付时间" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="入住人数">
        <el-input v-model="orderRoomType.manNum"></el-input>
      </el-form-item>
      <el-form-item label="入住日期">
        <el-input v-model="orderRoomType.inDate" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="离店日期">
        <el-input v-model="orderRoomType.outDate" readonly="readonly"></el-input>
      </el-form-item>

      <div class="info-split">会员信息</div>
      <el-form-item label="姓名">
        <el-input v-model="member.name" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="手机号">
        <el-input v-model="member.phone" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="身份证号">
        <el-input v-model="member.idCard" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="会员等级">
        <el-input v-model="memberLevel.lvName" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="金币余额">
        <el-input v-model="member.gold" placeholder="金币余额" readonly="readonly"></el-input>
      </el-form-item>
      <el-form-item label="积分余额">
        <el-input v-model="member.integral" controls-position="right" :min="0" label="排序号" readonly="readonly"></el-input>
      </el-form-item>

      <div v-if="order.orderType!=1">
        <div class="info-split">房型信息</div>
        <el-form-item label="房型名称">
          <el-input v-model="roomType.name" readonly="readonly"></el-input>
        </el-form-item>
        <el-form-item label="可住人数">
          <el-input v-model="roomType.manNum" readonly="readonly"></el-input>
        </el-form-item>
        <el-form-item label="标签">
          <el-input class="menu-list__input" v-model="roomType.tags" readonly="readonly"></el-input>
        </el-form-item>
        <el-form-item label="床型" prop="url">
          <el-input v-model="roomType.bed" placeholder="床型" readonly="readonly"></el-input>
        </el-form-item>
      </div>
      <div v-if="order.orderType!=1">
        <div class="info-split">价格信息</div>
        <el-form-item label="市场价" prop="perms" label-width="120px">
          <el-input placeholder="市场价" v-model="roomType.price"></el-input>
        </el-form-item>
        <el-form-item v-for="item in roomTypePrices" :label="item.memberLevel.lvName" label-width="120px">
          <el-input v-model="item.price" readonly="readonly"></el-input>
        </el-form-item>
      </div>
      <div v-if="order.orderType==1">
        <div class="info-split">主题套餐</div>
        <div style="display: inline-block;width: 150px;height: 100px;">
          <img :src="dispkg.img" width="100%" height="100%" style="border-radius:10px;margin-top:-40px">
        </div>
        <div style="display: inline-block;">
          <div>
            <el-form-item label="主题名称" prop="perms">
              <el-input placeholder="市场价" v-model="dispkg.name"></el-input>
            </el-form-item>
            <el-form-item label="价格">
              <el-input v-model="dispkg.price" readonly="readonly"></el-input>
            </el-form-item>
          </div>

          <div>
            <el-form-item label="套餐内容">
              <el-input type="textarea" v-model="dispkg.goods" style="width: 455px;"></el-input>
            </el-form-item>
          </div>
        </div>


      </div>

    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { treeDataTranslate } from '@/utils'
  import Icon from '@/icons'
  export default {
    data () {
      return {
        visible: false,
        order:{},
        member:{},
        memberLevel:{},
        orderRoomType:{},
        dispkg:{},
        roomType:{
          name:'',
          bed:'',
          tags:'',
          manNum:''
        }
      }
    },
    created () {

    },
    methods: {
      init (id) {
        this.order.id = id || 0
        this.$http({
          url: this.$http.adornUrl(`/od/order/getOne?orderId=`+id),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          console.log(data.order)
          var orderType = data.order.orderType
          if(orderType==0){
            data.order.orderTypeName = '普通房型订单'
          } else if(orderType==1){
            data.order.orderTypeName = '主题套餐订单'
          }else if(orderType==2){
            data.order.orderTypeName = '抢购房型订单'
          }else if(orderType==3){
            data.order.orderTypeName = '特价房型订单'
          }
          console.log(this.orderStatusConvert(data.order.status))
          this.order = data.order
          this.order.statusName = this.orderStatusConvert(data.order.status)
          this.member = data.order.member
          if(data.order.orderType!=1){
            var roomType = data.order.orderRoomType.roomType

            var tags = roomType.tags
            var tagsTxt = ''
            if(tags&&tags.length>0) {
              for(var i=0;i<tags.length;i++) {
                tagsTxt+=tags[i]+','
              }
              tagsTxt = tagsTxt.substr(0,tagsTxt.length-1)
              roomType.tags = tagsTxt
            }
            this.roomType = roomType
            this.orderRoomType = data.order.orderRoomType
            this.roomTypePrices = roomType.roomTypePrices
          }

          if(data.order.orderType==1){
            this.dispkg = data.order.orderPackage.discountPackage
          }

          this.memberLevel = data.order.member.memberLevel
        }).then(() => {
          this.visible = true
        })
      },
      orderStatusConvert(status){
        switch (status) {
          case 0:
            return '待支付'
            break;
          case 1:return '已支付'
            break;
          case 2: return '待入住'
            break;
          case 3: return '已完成'
            break;
          case 4: return '已取消'
            break;
          default: return '待支付'
        }
      }
    }
  }
</script>

<style lang="scss">
  .mod-menu {
    .menu-list__input,
    .icon-list__input {
      > .el-input__inner {
        cursor: pointer;
      }
    }
    &__icon-popover {
      max-width: 370px;
    }
    &__icon-list {
      max-height: 180px;
      padding: 0;
      margin: -8px 0 0 -8px;
      > .el-button {
        padding: 8px;
        margin: 8px 0 0 8px;
        > span {
          display: inline-block;
          vertical-align: middle;
          width: 18px;
          height: 18px;
          font-size: 18px;
        }
      }
    }
    .icon-list__tips {
      font-size: 18px;
      text-align: center;
      color: #e6a23c;
      cursor: pointer;
    }
  }
  .info-split{
    width: 100%;
    height: 30px;
    line-height: 30px;
    font-weight: bold;
    border-bottom: 1px dashed lightgray;
    margin-bottom: 10px;
  }
</style>
