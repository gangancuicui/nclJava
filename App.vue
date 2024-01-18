<script>
	import {
		mapMutations,
		mapState
	} from 'vuex'
	export default {
		...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateAddress']),
		onLaunch: function() {
			wx.checkSession({
				 success () {
				    console.log(true)
				  },
				  fail () {
					  uni.showToast({
					    title:"登录过期",
					    duration:"1500",
					    icon: 'none',
					  })
					  uni.setStorageSync('userinfo', null)
					  uni.setStorageSync('token', null)
					  
				  }
				})
		},
		globalData: {
		    data: {}
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style>
	/*每个页面公共css */
</style>
