<template>
	<view>
		<view class="container">
			<!-- 提示登录的图标 -->
			<uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
			<!-- 登录按钮 -->
			<button type="primary" class="login" @click="getUserInfo">一键登录</button>
			<!-- 登录提示 -->
			<view class="tips-text">登录后尽享更多权益</view>
		</view>
		<view class="popup-container" v-if="show">
			<view class="popup">
				<view>填写您的用户信息</view>
				<view>您的信息仅会被用于登录</view>

				<button class="avatar-wrapper" open-type="chooseAvatar" @chooseavatar="onChooseAvatar">
					<image :src="avatar"></image>
				</button>
				<view>点击选择头像</view>
				<form @submit="formSubmit">
					<view>
						<view>昵称</view>
						<input name="nickName" type="nickname" class="weui-input" v-model="nickname"
							placeholder="请输入昵称" />
					</view>
					<view class="btn">
						<button @click.native="close" class="btn">关闭</button>
						<button class="btn2" form-type="submit">提交</button>
						<!-- <button open-type="getPhoneNumber" @getphonenumber="getPhoneNumber">手机号</button> -->
					</view>
				</form>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		mapMutations,
		mapState
	} from 'vuex'
	export default {
		name: "my-login",
		data() {
			return {
				show: false,
				avatar: '/static/OIP.jpg',
				openid: '',
				code:"",
				encryptedData:"",
				iv:"",
				signature:""
				
			};
		},
		computed: {
			...mapState('m_user', ['redirectInfo']),
		},
		
		methods: {
			...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),
			onChooseAvatar(e) {
				console.log(e)
				this.avatar = e.detail.avatarUrl
				console.log(this.avatar)
				// wx.cloud.uploadFile({
				// 	cloudPath: "头像/" + Date.now() + ".jpeg",
				// 	filePath: this.avatar
				// }).then(res => {
				// 	console.log(res.fileID)
				// 	this.avatar = res.fileID
				// }).catch(error => {
				// 	uni.showToast({
				// 		title: "失败",
				// 		icon: 'error',
				// 		duration: 2000
				// 	})
				// })
				uni.uploadFile({
					url: 'http://127.0.0.1:7070/uploadavatar', 
					filePath: this.avatar,
					name: 'image',
					success: (uploadFileRes) => {
						console.log(uploadFileRes);
						let jsonString =uploadFileRes.data
						let startString = '"data":"';
						let endString = '"';
						// 找到起始字符串的位置
						let startIndex = jsonString.indexOf(startString);
						// 找到结束字符串的位置
						let endIndex = jsonString.indexOf(endString, startIndex + startString.length);
						 let result = jsonString.substring(startIndex + startString.length, endIndex);
						this.avatar=result
						console.log(this.avatar)
						uni.showToast({
							title: "成功",
							icon: 'success',
							duration: 2000
						})
					},fail: (err) => {
						uni.showToast({
							title: "失败",
							icon: 'error',
							duration: 2000
						})
					}
					
				});

			},
			async getUserProfile(){
				wx.getUserProfile({
				      desc: '用于完善会员资料', // 声明获取用户个人信息后的用途，后续会展示在弹窗中，请谨慎填写
				      success: (res) => {
				       console.log(res)
							this.encryptedData=res.encryptedData
							this.iv=res.iv
							this.signature=res.signature
				      }
				    }) 
			 },
			async formSubmit(e) {
				
				console.log(e)
				var useerInfo = {
					nickName: e.detail.value.nickName,
					avatarUrl: this.avatar,
					openid: this.openid,
					encryptedData:this.encryptedData,
					iv:this.iv,
					signature:this.signature,
					code:this.code
				}
				console.log(useerInfo)
				if(e.detail.value.nickName==""){
					return uni.$showMsg()
				}
				const {
					data: resLogin
				} = await uni.$http.post("/signup", useerInfo)

				if(resLogin == "注册失败"){
					return uni.$showMsg()
				}
				
				this.updateUserInfo(useerInfo)
				this.getToken(resLogin.data)
			},
			close() {
				this.show = false;
			},

			async login(code) {
				if (code) {
					const {
						data: resLogin
					} = await uni.$http.post("/login", {
						code: code,
					})
					console.log(1)
					console.log(resLogin.data)
					if (resLogin.msg == "success") {
						this.loginafter(resLogin.data)
					} else {
						this.openid=resLogin.data
						this.show = true
						
					}
				} else {
					console.log('登录失败！' + res.errMsg)
				}
			},

			getUserInfo(e) {
				this.getUserProfile()
				var _this = this
				if (e.detail.errMsg === 'getUserInfo:fail auth deny') {
					return uni.$showMsg('您取消了登录授权！')
				}
				uni.showModal({
					title: '温馨提示',
					content: '亲，授权微信登录后才能正常使用小程序功能',
					success: (res) => {
						//如果用户点击了确定按钮
						if (res.confirm) {
							wx.login({
								success(suc) {
									_this.code=suc.code
									_this.login(suc.code)
									
									
								}
							})
						} else if (res.cancel) {
							//如果用户点击了取消按钮
							uni.showToast({
								title: '您拒绝了请求,不能正常使用小程序',
								icon: 'error',
								duration: 2000
							});
							return;
						}
					}
				});
			},
			loginafter(data) {
				var useerInfo = {
					nickName: data.nickName,
					avatarUrl: data.avatarUrl,
					openid: data.openid
				}
				this.openid=data.openid
				this.updateUserInfo(useerInfo)
				this.getToken(data.token)
			},
			async onSaveUserInfo(e) {
				const {
					userInfo,
					encryptedData,
					rawData,
					iv,
					signature
				} = e;

				//通过解构赋值提取出 code (验证码)
				const [, {
					code
				}] = await uni.login();

				// 使用拿到的数据发送请求获取token
				const res = await this.$u.api.getUserToken({
					encryptedData,
					rawData,
					iv,
					signature,
					code,
				});
				if (res.message) {
					// 提取出 token
					const {
						token
					} = res.message;
					// 把 token 保存到本地
					uni.setStorageSync("token", token);
					// 获取到token 才渲染用户头像
					this.userInfo = userInfo;
					uni.setStorageSync("userInfo", userInfo);
				} else {
					// 登录错误提示
					uni.showToast({
						title: "登录错误，请重试",
						icon: "none"
					});
				}
			},
			async getToken(token) {
				// const res = await uni.login()
				// const query = {
				// 	code: res.code,
				// 	encryptedData: info.encryptedData,
				// 	iv: info.iv,
				// 	rawData: info.rawData,
				// 	signature: info.signature
				// }
				uni.$showMsg('登录成功')
				this.navigateBack()
				this.updateToken(token)
			},
			navigateBack() {
				if (this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
					uni.switchTab({
						url: this.redirectInfo.from,
						complete: () => {
							this.updateRedirectInfo(null)
						}
					})
				}
			}


		}
	}
</script>

<style lang="scss">
	.container {
		// 登录盒子的样式
		height: 750rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background-color: #f8f8f8;
		position: relative;
		overflow: hidden;

		// 绘制登录盒子底部的半椭圆造型
		&::after {
			content: ' ';
			display: block;
			position: absolute;
			width: 100%;
			height: 40px;
			left: 0;
			bottom: 0;
			background-color: white;
			border-radius: 100%;
			transform: translateY(50%);
		}

		// 登录按钮的样式
		.login {
			width: 90%;
			border-radius: 100px;
			margin: 15px 0;
			background-color: #00aa00;
		}

		// 按钮下方提示消息的样式
		.tips-text {
			font-size: 12px;
			color: gray;
		}
	}


	.popup-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.5);
		z-index: 999;
	}

	.popup {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		width: 80%;
		max-width: 500px;
		background-color: #fff;
		border-radius: 10px;
		padding: 20px;
		box-sizing: border-box;
	}

	input {
		height: 100rpx;
	}

	.popup view:first-child {
		font-size: 18px;
		font-weight: bold;
		margin-bottom: 10px;
	}

	.popup view:nth-child(2) {
		font-size: 14px;
		color: #999;
		margin-bottom: 20px;
	}

	.popup .avatar-wrapper {
		display: block;
		margin: 0 auto;
		width: 80px;
		height: 80px;
		border-radius: 50%;
		overflow: hidden;
		margin-bottom: 10px;
	}

	.popup image {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}

	.popup view:nth-child(4) {
		font-size: 14px;
		color: #999;
		text-align: center;
		margin-bottom: 20px;
	}

	.popup input[type="nickname"] {
		width: 100%;
		padding: 10px;
		box-sizing: border-box;
		border: 1px solid #ccc;
		border-radius: 5px;
		margin-bottom: 20px;
	}

	.popup .btn {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.popup .btn button {
		background-color: #fff;
		border: 1px solid #ccc;
		border-radius: 5px;
		padding: 10px 20px;
		box-sizing: border-box;
		color: #333;
		font-size: 14px;
	}

	.popup .btn button.btn2 {
		background-color: #1aad19;
		color: #fff;
		border-color: #1aad19;
	}
</style>