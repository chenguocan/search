<script>
	export default {
		globalData:{
			baseUrl:'http://81.69.177.60:8090',
			openid:'',
			register:0,
			err:'',
			err_count:'',
			userInfo:'',
		},
		onLaunch: async function(options) {
			await this.getOpenid();
			if(options.query.id){
				let id=options.query.id;
				uni.redirectTo({
					url:'/pages/search/search?id='+id
				})
			}
		},
		onShow: function() {
			
		},
		onHide: function() {
			
		},
		methods:{
			getOpenid(){
				let that=this;
				// #ifdef MP-WEIXIN
					wx.login({
						success(res){
							console.log(res);
							if(res.code){
								uni.request({
									url:that.globalData.baseUrl+'/user/getOpenId',
									method:"POST",
									data:{
										code:res.code
									},
									success(result) {
										if(result.data.errCode===0){
											that.globalData.openid=result.data.data.data.openid;
											that.getRegister(that.globalData.openid);
										}
									}
								})
							}
						},
						fail(err) {
							uni.showToast({
								tite:'获取登录信息失败',
								icon:'none'
							})
						}
					})
				// #endif
			},
			getRegister(openid){
				let that=this;
				uni.request({
					method:'POST',
					url:that.globalData.baseUrl+'/user/getRegister',
					data:{
						openid,
					},
					success(res) {
						if(res.data.errCode===0){
							that.globalData.register=res.data.data.register;
							that.getError(openid)
						}
					}
				})
			},
			getError(openid){
				let that=this;
				uni.request({
					method:'POST',
					url:that.globalData.baseUrl+'/user/getError',
					data:{
						openid,
					},
					success(res) {
						console.log(res);
						if(res.data.errCode===0){
							that.globalData.err=res.data.data.err;
							that.globalData.err_count=res.data.data.err_count;
							if(that.globalData.err && (parseInt(that.globalData.err_count%3)===0)){
								uni.showModal({
									title:'提示',
									content:'账号违纪次数过多，请联系管理员解封'
								})
								that.globalData.register=0
							}
							if(that.globalData.err){
								uni.showModal({
									title:'提示',
									content:that.globalData.err
								})
							}
							
							that.$isResolve();
						}
					}
				})
			}
		}
	}
</script>

<style lang="scss">
	@import "uview-ui/index.scss";
	@import "./static/iconfont/iconfont.css";
	/*每个页面公共css */
</style>
