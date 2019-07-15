<template>
  <div>
    <div class="goods-list-container">
      <Alert show-icon class="tips-box">
        小提示
        <Icon type="ios-lightbulb-outline" slot="icon"></Icon>
        <template slot="desc">请点击商品前的选择框，选择购物车中的商品，点击付款即可。</template>
      </Alert>

      <Table
        border
        ref="selection"
        :columns="columns"
        :data="userOrders"
        size="large"
        @on-selection-change="select"
        no-data-text="您的购物车没有商品，请先添加商品到购物车再点击购买"
      ></Table>
      <div class="address-container">
        <h3>收货人信息</h3>
        <div class="address-box">
          <div class="address-check">
            <div class="address-check-name">
              <span>
                <Icon type="ios-checkmark-outline"></Icon>
                {{checkAddress.name}}
              </span>
            </div>
            <div class="address-detail">
              <p>{{checkAddress.address}}</p>
            </div>
          </div>
        </div>
      </div>

      <div class="pay-container">
        <div class="pay-box">
          <p>
            <span>提交订单应付总额：</span>
            <span class="money">
              <Icon type="social-yen"></Icon>
              {{totalPrice.toFixed(2)}}
            </span>
          </p>
          <div class="pay-btn">
            <Button type="error" size="large" @click="buyMultiOrders()">支付订单</Button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Search from '@/components/Search';
import GoodsListNav from '@/components/nav/GoodsListNav';
import store from '@/vuex/store';
import { mapState, mapActions } from 'vuex';
export default {
  name: 'Order',
  beforeRouteEnter (to, from, next) {
    window.scrollTo(0, 0);
    next();
  },
  created () {
    this.loadAddress();
  },
  data () {
    return {
      userOrders: [],
      goodsCheckList: [],
      columns: [
        {
          type: 'selection',
          width: 58,
          align: 'center'
        },
        {
          title: '图片',
          key: 'img',
          width: 86,
          render: (h, params) => {
            return h('div', [
              h('img', {
                attrs: {
                  src: params.row.img
                }
              })
            ]);
          },
          align: 'center'
        },
        {
          title: '描述',
          key: 'info',
          align: 'center'
        },
        {
          title: '商品',
          key: 'productName',
          align: 'center'
        },
        {
          title: '数量',
          key: 'productAmount',
          width: 68,
          align: 'center'
        },
        {
          title: '价格',
          width: 68,
          key: 'price',
          align: 'center'
        },
        {
          title: '操作',
          width: 68,
          key: 'price',
          align: 'center',
          render: (h, params) => {
            
            return h('div', [
                h('Button', {
                    props: {
                      type: 'error',
                      size: 'small'
                    },
                    style: {
                       
                      fontSize: '5px'
                    },

                    on: { click: () => {
                      console.log(params.row.orderId);
                      this.deleteOrder(params.row.orderId);
                    }

                  }

              }, '删除'),

            ])
        }

        }
      ],
      checkAddress: {
        name: '未选择',
        address: '请选择地址'
      },
      remarks: ''
    };
  },
  computed: {
    ...mapState(['address', 'shoppingCart']),
    totalPrice () {
      let price = 0;
      this.goodsCheckList.forEach(item => {
        price += item.price * item.productAmount;
      });
      return price;
    }
  },
  methods: {
    ...mapActions(['loadAddress']),
    getAllOrders(userId){
      this.$axios({
        method: "get",
        url: 'http://localhost:10086/order/ordersByUserId?userId=' + userId,
        data: {

        }

      }).then(res=>{
        
        this.userOrders = res.data;
      })

    },
    select (selection, row) {
      console.log(selection);
      this.goodsCheckList = selection;
    },
    buyMultiOrders(){
      var orderIds = '';
      this.goodsCheckList.forEach(item => {
        orderIds += item.orderId + ',';
      });
      this.$axios({
        method: "post",
        url: 'http://localhost:10086/paid/paid?orderIds=' + orderIds,
        data: {

        }

      }).then(res=>{
        console.log(res.data);
        alert(res.data.msg);

      });

    },
    deleteOrder(orderId){
      console.log(orderId);
      this.$axios({
        method: "post",
        url: 'http://localhost:10086/order/orderDelete?id=' + orderId,
        data: {

        }

      }).then(res=>{
        console.log(res.data);
        this.getAllOrders(1);
      })


    }
  },
  mounted () {
    this.getAllOrders(1);
  },
  components: {
    Search,
    GoodsListNav
  },
  store
};
</script>

<style scoped>
.goods-list-container {
  margin: 15px auto;
  width: 80%;
}
.tips-box {
  margin-bottom: 15px;
}
.address-container {
  margin-top: 15px;
}
.address-box {
  margin-top: 15px;
  padding: 15px;
  border: 1px #ccc dotted;
}
.address-check {
  margin: 15px 0px;
  height: 36px;
  display: flex;
  align-items: center;
}
.address-check-name {
  width: 120px;
  display: flex;
  justify-content: center;
  align-content: center;
  background-color: #ccc;
}
.address-check-name span {
  width: 120px;
  height: 36px;
  font-size: 14px;
  line-height: 36px;
  text-align: center;
  color: #fff;
  background-color: #f90013;
}
.address-detail {
  padding-left: 15px;
  font-size: 14px;
  color: #999999;
}
.remarks-container {
  margin: 15px 0px;
}
.remarks-input {
  margin-top: 15px;
}
.invoices-container p {
  font-size: 12px;
  line-height: 30px;
  color: #999;
}
.pay-container {
  margin: 15px;
  display: flex;
  justify-content: flex-end;
}
.pay-box {
  font-size: 18px;
  font-weight: bolder;
  color: #495060;
}
.money {
  font-size: 26px;
  color: #f90013;
}
.pay-btn {
  margin: 15px 0px;
  display: flex;
  justify-content: flex-end;
}
</style>
