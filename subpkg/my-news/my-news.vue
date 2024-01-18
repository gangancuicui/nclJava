<template>
	<view class="container">
		<view class="top">
			<scroll-view class="scroll" scroll-x="true">
				<view class="tabs">
					<view :class="flag1 === '选种' ? 'tab active' : 'tab'" id="选种" @click="select"><text>选种</text></view>
					<view :class="flag1 === '种植' ? 'tab active' : 'tab'" id="种植" @click="select"><text>种植</text></view>
					<view :class="flag1 === '销售' ? 'tab active' : 'tab'" id="销售" @click="select"><text>销售</text></view>
				</view>
			</scroll-view>
		</view>
		<view>
			<view class="news-list">
				<view class="news-item" v-for="(item, index) in newsList" :key="index" >
					<view @click="goToDetail(item.id,item.title)">
						<view class="image">
							<image :src="item.img" mode="aspectFill" />
						</view>
						<view class="info">
							<text class="date">发布时间：{{item.ctime}}</text>
							<text class="title">{{ item.title }}</text>
							<text class="author">作者：{{item.author}}</text>
						</view>
					</view>
					<button class="del" @click="delById(item.id,item.paperid)">删除</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	// import badgeMix from '@/mixins/tabbar-badge.js'
	export default {
		// mixins: [badgeMix],
		data() {
			return {
				newsList: [],
				flag1: '种植',
				page: 1,
				size: 5,
				loading: false,
				total: 0,
				flag: false,
				openid:"",

				options: [{
					text: '删除',
					style: {
						backgroundColor: '#c8cac9'
					}
				}]
			};
		},
		onLoad(opentions) {
			this.openid = opentions.openid
			this.page = 1,
			this.size = 5,
				this.getNewsList()
		},
		onReachBottom() {
			 if (this.page * this.size >= this.total) {
				 wx.showLoading({
				       title: '数据加载完毕！',
				})
				wx.hideLoading(); // 关闭loading
				return ;
			 }
			if(this.loading)return;
			this.page++;
			this.flashNewsList()
		},

		methods: {
			goToDetail(id, title) {
				console.log(id)
				uni.navigateTo({
					url: '/subpkg/news-detail/news-detail?id=' + id + '&title=' + title

				});
			},
			async getNewsList() {
				this.flag = true;
				wx.showLoading({
					title: '数据加载中...',
				})
				const {
					data: res
				} = await uni.$http.get("/paper/self", {
					type: "种植",
					page: this.page,
					size: this.size,
					openid: this.openid
				})
				console.log(res)
				this.newsList = res.data.rows;
				this.total = res.data.total;
				wx.hideLoading();
				this.flag = false;
				console.log(res.data.rows);

			},
			async flashNewsList() {
				this.flag = true;
				wx.showLoading({
					title: '数据加载中...',
				})
				// const db =uni.cloud.database().collection('farmer').where({paperid:"种植"})
				//     db.get({
				//       success: (res)=>{
				//         console.log(res)
				//           this.newsList=[...this.newsList,...res.data]
				//         console.log(this.newsList)

				//       }
				//     })
				const {
					data: res
				} = await uni.$http.get("/paper/self", {
					type: this.flag1,
					page: this.page,
					size: this.size,
					openid: this.openid
				})
				console.log(res)
				this.newsList = res.data.rows;
				this.total = res.data.total;
				wx.hideLoading();
				this.flag = false;
				console.log(res.data.rows);

			},
			async refresh() {
				const {
					data: res
				} = await uni.$http.get("/paper/self", {
					type: this.flag1,
					page: this.page,
					size: this.size,
					openid: this.openid
				})
				console.log(res)
				this.newsList = res.data.rows
				console.log(res.data.rows);

			},
			async select(e) {
				console.log(e)
				var id = e.currentTarget.id
				this.flag1 = e.currentTarget.id
				this.page = 1;
				const {
					data: res
				} = await uni.$http.get("/paper/self", {
					type: id,
					page: this.page,
					size: this.size,
					openid: this.openid
				});
				console.log(res);
				this.newsList = res.data.rows
				this.total = res.data.total;
				console.log(this.newsList);
			},
			async delById(id, pid) {
				var _this = this;
				wx.showModal({
					title: '提示',
					content: '是否确认删除',
					async success(res) {
						if (res.confirm) {
							console.log('用户点击确定')
							console.log(id)
							const {
								data: res
							} = await uni.$http.post("/delPaper", {
								id:id,
								openid: _this.openid
							});
							if(res.code!=1){
								uni.$showMsg("错误")
							}
							uni.$showMsg('删除成功')
							_this.flashNewsList()

						} else if (res.cancel) {
							console.log('用户点击取消')
						}
					}
				})

			}
		}
	}
</script>

<style lang="scss">
	.container {
		padding: 10rpx;
		background-color: #f0f8f5;
	}

	.top {
		height: 88rpx;
		color: #ffffff;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.scroll {
		width: 100%;
		height: 60rpx;
	}

	.tabs {
		display: flex;
		flex-wrap: nowrap;
		justify-content: flex-start;
		align-items: center;
		height: 100%;
		padding-left: 20rpx;
	}

	.tab {
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 32rpx;

		text {
			color: black;
		}

		padding: 12rpx 0;
		border-radius: 20rpx;
		color: #ffffff;
		margin-right: 20rpx;
		background-color: transparent;
		transition: background-color 0.3s;
	}

	.tab.active {
		background-color: rgb(0, 170, 0);
		color: #00aa00;

		text {
			color: #fff;
		}
	}

	.news-list {
		display: flex;
		flex-direction: column;
		margin-top: 20rpx;
	}

	.news-item {
		display: flex;
		align-items: flex-start;
		margin-bottom: 20rpx;
		padding: 20rpx;
		background-color: #ffffff;
		border-radius: 10rpx;
		box-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.1);
		transition: transform 0.3s, box-shadow 0.3s;
		position: relative;
		/* 添加这个 */
	}

	.news-item:hover {
		transform: translateY(-5rpx);
		box-shadow: 0 4rpx 6rpx rgba(0, 0, 0, 0.1);
	}

	.image {
		width: 200rpx;
		height: 200rpx;
		margin-right: 20rpx;
		overflow: hidden;
		border-radius: 10rpx;
	}

	.image image {
		width: 100%;
		height: 100%;
		display: block;
		object-fit: cover;
	}

	.info {
		flex: 1;


	}

	.date {
		font-size: 24rpx;
		color: #74c58e;
		position: absolute;
		bottom: 15rpx;
		right: 20rpx;
	}

	.author {
		font-size: 24rpx;
		color: #74c58e;
		position: absolute;
		bottom: 15rpx;
		left: 240rpx;
	}

	.title {
		position: absolute;
		left: 250rpx;
		top: 10rpx;
		font-size: 36rpx;
		font-weight: bold;
		margin-bottom: 10rpx;
		color: #333333;
	}

	.del {
		position: absolute;
		color: #fff;
		left: 550rpx;
		top: 60rpx;
		background-color: #db7b7b;


	}
</style>