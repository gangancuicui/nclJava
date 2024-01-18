<template>
  <view>
    <view class="address">
      <text class="title">收货人：{{orderInfo.address.userName}}&nbsp;电话：{{orderInfo.address.telNumber}}</text>
      <view class="info">地址：{{orderInfo.address.provinceName+orderInfo.address.cityName+orderInfo.address.countyName+orderInfo.address.detailInfo }}</view>
    </view>
    <view class="order">
      <view class="num">订单号: {{ orderInfo.orderNumber }}</view>
      <view class="goods">
        <view class="goods-item" v-for="(item, i) in orderInfo.orderDetail" :key="i">
          <view class="logo">
            <img :src="item.goodsLogo" class="goods-image">
          </view>
          <view class="details">
            <view class="name">商品名称: {{ item.goodsName }}</view>
            <view class="nums">数量: {{ item.goodsNumber }}</view>
            <view class="price">单价: ￥{{ item.goodsPrice }}</view>
          </view>
        </view>
      </view>
      <view class="count">
        <view class="count-price">订单总价：￥{{ orderInfo.orderCount }}</view>
        <view class="time">订单创建时间：{{ orderInfo.creationTime }}</view>
        <view class="state1">
			<view class="state">订单状态：</view>
			<view class="state" v-if="(orderInfo.orderStatus==1)">未支付</view>
			<view class="state" v-if="(orderInfo.orderStatus==2)">已支付待发货</view>
			<view class="state" v-if="(orderInfo.orderStatus==3)">已发货</view>
			<view class="state" v-if="(orderInfo.orderStatus==4)">订单完成</view>
		</view>
		<view class="state1">
			<view class="state">快递单号：</view>
			<view class="state" v-if="(!orderInfo.expressNumber)">未发货</view>
			<view class="state" v-if="(orderInfo.expressNumber)">{{orderInfo.expressNumber}}</view>
		</view>
		
        <view class="pay" v-if="(orderInfo.orderStatus==1)">
          <button type="primary" @click="pay" id=1>支付</button>
        </view>
		<view class="pay" v-if="(orderInfo.orderStatus==2)">
		  <button type="primary" @click="pay" id=2>催发货</button>
		</view>
		<view class="pay" v-if="(orderInfo.orderStatus==3)">
		  <button type="primary" @click="pay" id=3>确认收货</button>
		</view>
		<view class="pay" v-if="(orderInfo.orderStatus==4)">
		  <button type="primary" open-type="contact">申请售后</button>
		</view>
      </view>
    </view>
  </view>
</template>

<script>
import { $http } from '@escook/request-miniprogram';
import {
		mapGetters,
		mapMutations,
		mapState
	} from 'vuex';
	export default {
		data() {
			return {
				orderInfo:{}
			};
		},
		computed:{
			...mapState('m_user', ['userinfo']),
		},
		onLoad(options) {
			// let db=uni.cloud.database().collection('order').where({_id:options._id})
			// db.get({
			// 	success :(res)=>{
			// 		console.log(res.data[0])
			// 		this.orderInfo=res.data[0]
			// 		console.log(this.orderInfo)
			// 	}
			// })
			this.getOrderDetail(options.id)
			
		},
		methods:{
			async pay(e){
				console.log(e)
				if(e.currentTarget.id==1){
					// const db=wx.cloud.database()
					// db.collection('order').where({
					//       _id: this.orderInfo._id
					//     })
					//     .update({
					//       data: {
					//         order_state:true
					//       },
					// 	  success:(r)=>{
					// 	  	wx.showToast({
					// 	  		title: '支付成功',
					// 	  		duration: 3000,
					// 	  		success: function () {
					// 	  			setTimeout(function () {
					// 	  	        wx.navigateBack({
					// 	  	          	 delta: 1
					// 	  	        })
					// 	  	            }, 1000);
					// 	  	         }
					// 	  	})
					// 	  }
					// })
					const {data:res}
					= await $http.post("/order/updateOrder",{
						id:this.orderInfo.id,
						openid:this.orderInfo.openid,
						orderStatus:2
					})
					if(res.code !=1){
						wx.showToast({
							title: '支付失败',
							duration: 3000,
							
						})
					}
					else{
						wx.showToast({
							title: '支付成功',
							duration: 3000,
							success: function () {
								setTimeout(function () {
						        wx.navigateBack({
						          	 delta: 1
						        })
						            }, 1000);
						         }
						})
					}
					
					
				}else if(e.currentTarget.id==2){
					// const db=wx.cloud.database()
					// db.collection('order').where({
					//       _id: this.orderInfo._id
					//     })
					//     .update({
					//       data: {
					//         payafter:true
					//       },
					// 	  success:(r)=>{
					// 		  wx.showToast({
					// 			title: '收货成功',
					// 			duration: 3000,
					// 			success: function () {
					// 				setTimeout(function () {
					// 		        wx.navigateBack({
					// 		          	 delta: 1
					// 		        })
					// 		            }, 1000);
					// 		         }
					// 		})
					// 	  }
					// })
					
					wx.showToast({
						title: '催发货成功',
						duration: 3000,
					})
					
					
				}else if(e.currentTarget.id==3){
					const {data:res}
					= await $http.post("/order/updateOrder",{
						id:this.orderInfo.id,
						openid:this.orderInfo.openid,
						orderStatus:4
					})
					if(res.code !=1){
						wx.showToast({
							title: '收货成功',
							duration: 3000,
							
						})
					}
					else{
						wx.showToast({
							title: '收货成功',
							duration: 3000,
							success: function () {
								setTimeout(function () {
						        wx.navigateBack({
						          	 delta: 1
						        })
						            }, 1000);
						         }
						})
					}
				}
				
				
			},
			async getOrderDetail(id){
				const {data:res}=await $http.get("/order/getOrder/"+id);
					
				console.log(res)
				
				this.orderInfo=res.data
				
			}
		}
	}
</script>

<style lang="scss">
  .address {
    margin-bottom: 20px;
  }

  .title {
    font-size: 18px;
    font-weight: bold;
  }

  .info {
    font-size: 16px;
    color: #333;
    margin-top: 10px;
  }

  .order {
    border: 1px solid #ccc;
    padding: 20px;
  }

  .num {
    font-size: 36rpx;
    font-weight: bold;
  }

  .goods-item {
    display: flex;
    align-items: center;
    margin-top: 15px;
  }

  .logo img {
    width: 80px; 
    height: 80px; 
  }

  .details {
    margin-left: 15px;
  }
  .state1{
	  display: flex;
  }

  .name {
    font-size: 16px;
    color: #333;
	white-space: pre-wrap;
  }

  .nums,
  .price {
    font-size: 14px;
    color: #666;
	white-space: pre;
  }

  .count-price {
    font-size: 18px;
    font-weight: bold;
  }

  .time,
  .state {
    font-size: 16px;
    color: #666;
  }

  .pay {
    margin-top: 20px;
  }

  .pay button {
    padding: 10px 20px;
    font-size: 16px;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
</style>
