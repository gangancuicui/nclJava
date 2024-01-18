<template>
	<view>
		<view class="tabs">		  
		  <view :class="flag === '待付款' ? 'tab active' : 'tab'" id="1" @click="select"><text>待付款</text></view>
		  <view :class="flag === '待发货' ? 'tab active' : 'tab'" id="2" @click="select"><text>待收货</text></view>
		  <view :class="flag === '待收货' ? 'tab active' : 'tab'" id="3" @click="select"><text>待收货</text></view>
		  <view :class="flag === '退款/售后' ? 'tab active' : 'tab'" id="4" @click="select"><text>售后</text></view>
		  <view :class="flag === '全部订单' ? 'tab active' : 'tab'" id="5" @click="select"><text>全部</text></view>
		</view>
		<view class="order-list">
			<view class="order-item" v-for="(item,i) in orderInfo" :key="i" @click="orderDetail(item.id)">
				<view class="num">订单号：</view>
				<text>{{item.orderNumber}}</text>
				<view class="num">订单价格：</view>
				<text>￥{{item.orderCount}}</text>
				<view class="num">订单创建时间：</view>
				<text>{{item.creationTime}}</text>
			</view>
		</view>
	</view>
</template>

<script>
	import { $http } from '@escook/request-miniprogram';
import { mapState, mapMutations } from 'vuex'
	export default {
		
		data() {
			return {
				orderInfo:[],
				flag:''
			};
		},
		computed:{
			...mapState('m_user', ['userinfo']),
		},
		onLoad(optins) {
			if(optins.query==1){
				this.flag='待付款'
				this.getorderInfo1()
			}
			else if(optins.query==2){
				this.flag='待发货'
				this.getorderInfo2()
			}
			else if(optins.query==3){
				this.flag='待收货'
				this.getorderInfo3()
			}
			else if(optins.query==4){
				this.flag='退款/售后'
				this.getorderInfo4()
			}
			else if(optins.query==5){
				this.flag='全部订单'
				this.getorderInfo5()
			}
			
		},
		methods:{
			async getorderInfo4(){
				const {data:res}=await $http.get("/order/getOrderList",{
					openid:this.userinfo.openid,
					status:4
				})
				if(res.code==0){
					return uni.$showMsg()
				}
				this.orderInfo=res.data
				// let db=uni.cloud.database().collection('order').where({_openid:this.userinfo.openid})
				// db.get({
				// 	success :(res)=>{
				// 		console.log(res.data)
				// 		this.orderInfo=res.data
				// 	}
				// })
			},
			async getorderInfo1(){
				const {data:res}=await $http.get("/order/getOrderList",{
					openid:this.userinfo.openid,
					status:1
				})
				if(res.code==0){
					return uni.$showMsg()
				}
				this.orderInfo=res.data
				// let db=uni.cloud.database().collection('order').where({_openid:this.userinfo.openid})
				// db.get({
				// 	success :(res)=>{
				// 		console.log(res.data)
				// 		this.orderInfo=res.data
				// 	}
				// })
			},
			async getorderInfo2(){
				const {data:res}=await $http.get("/order/getOrderList",{
					openid:this.userinfo.openid,
					status:2
				})
				if(res.code==0){
					return uni.$showMsg()
				}
				this.orderInfo=res.data
				// let db=uni.cloud.database().collection('order').where({_openid:this.userinfo.openid})
				// db.get({
				// 	success :(res)=>{
				// 		console.log(res.data)
				// 		this.orderInfo=res.data
				// 	}
				// })
			},
			async getorderInfo3(){
				const {data:res}=await $http.get("/order/getOrderList",{
					openid:this.userinfo.openid,
					status:3
				})
				if(res.code==0){
					return uni.$showMsg()
				}
				this.orderInfo=res.data
				// let db=uni.cloud.database().collection('order').where({_openid:this.userinfo.openid})
				// db.get({
				// 	success :(res)=>{
				// 		console.log(res.data)
				// 		this.orderInfo=res.data
				// 	}
				// })
			},
			async getorderInfo5(){
				const {data:res}=await $http.get("/order/getOrderAll",{
					openid:this.userinfo.openid,
				})
				if(res.code==0){
					return uni.$showMsg()
				}
				this.orderInfo=res.data
				// let db=uni.cloud.database().collection('order').where({_openid:this.userinfo.openid})
				// db.get({
				// 	success :(res)=>{
				// 		console.log(res.data)
				// 		this.orderInfo=res.data
				// 	}
				// })
			},
			
			select(e){
			    console.log(e)
			    var id=e.currentTarget.id
				this.flag=e.currentTarget.id
			    if(id==1){
					this.flag='待付款'
					this.getorderInfo1()
				}
				if(id==2){
					this.flag='待发货'
					this.getorderInfo2()
				}
				if(id==3){
					this.flag='待收货'
					this.getorderInfo3()
				}
				if(id==4){
					this.flag='退款/售后'
					this.getorderInfo4()
				}
				if(id==5){
					this.flag='全部订单'
					this.getorderInfo5()
				}
				
			},
			orderDetail(id){
				uni.navigateTo({
					url: '/subpkg/order-detail/order-detail?id='+ id
				});
			}
		}
	}
</script>

<style lang="stylus">
 /* 标签栏样式 */
.tabs {
  display: flex;
  flex-wrap: nowrap;
  justify-content: flex-start;
  align-items: center;
  height: 100%;
  // padding-left: 20rpx;
  margin: 10rpx;
}


.tab {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32rpx;
  text{
	  color: black;
  }
  padding: 12rpx 0;
  border-radius: 20rpx;
  color: #ffffff;
  margin:0 10rpx;
  background-color: transparent;
  transition: background-color 0.3s;
}

.tab.active {
  background-color: rgb(0,170,0);
  color: #00aa00;
  text{
  	  color: #fff;
  }
}

  .underline {
    position: absolute;
    left: 0;
    bottom: 0;
    height: 2px;
    background-color: #007bff;
    transition: width 0.3s ease;
  }

  /* 订单列表项样式 */
  .order-item {
    padding: 15px;
    border-bottom: 1px solid #ccc;
    background-color: #fff;
  }

  .num {
    font-size: 14px;
    color: #333;
	 font-weight: bold;
  }

  text {
    font-size: 14px;
    color: #666;
  }
</style>
