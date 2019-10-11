<template>
  <div class="mod-home">
    <div class="order-panel">
      <div style="display: inline-block;">
        <div class="tag-item" >
          <div>
            <img src="../../assets/img/dk_od.png" style="width: 80px;">
          </div>
          <div class="tag-txt">
            <div style="text-align: center;font-size: 32px;font-weight: 500;">{{orderData.day}}</div>
            <div style="text-align: center;color: #333;">今日订单数</div>
          </div>
        </div>
      </div>

      <div style="display: inline-block;">
        <div class="tag-item" >
          <div >
            <img src="../../assets/img/dk_day_num.png" style="width: 80px;">
          </div>
          <div class="tag-txt">
            <div style="text-align: center;font-size: 32px;font-weight: 500;">{{saleData.dayNum}}</div>
            <div style="text-align: center;color: #333;">今日销售额</div>
          </div>
        </div>
      </div>
      <div style="display: inline-block;">
        <div class="tag-item" >
          <div>
            <img src="../../assets/img/dk_ys_num.png" style="width: 80px;">
          </div>
          <div class="tag-txt">
            <div style="text-align: center;font-size: 32px;font-weight: 500;">{{saleData.monthNum}}</div>
            <div style="text-align: center;color: #333;">昨日销售额</div>
          </div>
        </div>
      </div>
    </div>

    <hr style="border:1px dashed #dcdfe6;margin: 20px 0;"/>

  </div>
</template>

<style>
  .order-panel{
    width: 90%;
    margin: 0 auto;
    display: flex;
    flex-direction: row;
    justify-content: center;
  }
  .tag-item{
    display: flex;
    flex-direction: row;
    border:1px solid #dcdfe6;
    width: 250px;
    align-items: center;
    justify-content: center;
    padding: 20px 0;height: 140px;
    margin-right: 30px;
  }
  .tag-txt{
    margin-left: 10px;
  }
</style>
<script>
  export default {
    data() {
      return {
        orderData:{},
        saleData:{}
      }
    },
    created(){
      this.getHomeData()
    },
    methods: {
      getHomeData(){
        this.$http({
          url: this.$http.adornUrl('/od/order/orderSum'),
          method: 'get'
        }).then(({data}) => {
          console.log('getHomeData')
          console.log(data)
          this.orderData = data.orderSum.orderData
          this.saleData = data.orderSum.saleData
        })
      }
    }
  }
</script>

<style>
  .mod-home {
    line-height: 1.5;
  }
</style>

