<template>
	<view class="my-page">
		<view style="height: 20rpx;"></view>
		<view class="background">
			<view class="myDetail" v-if="userInfo">
				<u-avatar :src="userInfo.avatar" @click="toDetail"></u-avatar>
				<view style="margin-left: 10rpx;" v-if="userInfo">{{userInfo.name}}</view>
			</view>
			<view class="myDetail" v-else @click="toRegister">
				<u-avatar src=""></u-avatar>
				<view style="margin-left: 10rpx;">请登陆</view>
			</view>
		</view>
		<view class="menu">
			<view class="header">
				<text style="margin-left: 20rpx;">我的菜单</text>
			</view>
			<view class="menu-item" v-for="(item,index) in menuList" :key="index" @click="toNav(item.index)">
				<view class="nav">
					<view class="iconfont" :class="item.icon" style="color:#07d165 ;">
					</view>
					<text style="margin-left: 30rpx;">{{item.title}}</text>
				</view>
				<view class="iconfont icon-you"></view>
			</view>
			<view class="menu-item">
				<view class="nav">
					<view class="iconfont icon-kefu" style="color:#07d165 ;">
					</view>
					<button open-type="contact" hover-class='none'
						style="font-weight: normal;font-size: 30rpx;text-align: start;width: 600rpx;">联系客服</button>
				</view>
				<view class="iconfont icon-you"></view>
			</view>
		</view>
		<u-popup v-model="show" mode="center" border-radius="15" @close="closePopup" width="500rpx" height="600rpx" closeable >
			<view style="height: 100rpx;"></view>
			<view style="display: flex;justify-content: center;align-items: center;flex-direction: column;">
				<u-avatar :src="form.avatar" @click="chooseAvatar"></u-avatar>
				
				<u-form :model="form" ref="uForm">
					<u-form-item label="姓名" prop="name">
						<u-input v-model="form.name" :clearable="false" />
					</u-form-item>
				</u-form>
				
				<u-button size="medium" type="primary" style="margin-top: 100rpx;" @click="updateInfo">修改</u-button>
			</view>

		</u-popup>
	</view>
</template>

<script>
	const globalData = getApp().globalData;
	export default {
		data() {
			return {
				form:{
					name:'',
					avatar:'',
					avatarValue:'',
				},
				userInfo: '',
				show: false,
				menuList: [{
					title: '创建问卷',
					index: 1,
					icon: 'icon-chuangjianwenjuan'
				}, {
					title: '官方模板',
					index: 2,
					icon: 'icon-moban'
				}, {
					title: '问卷历史',
					index: 3,
					icon: 'icon-tongji'
				}]
			}
		},
		async onLoad() {
			await this.$onLaunched;
			this.getUser();
		},
		methods: {
			closePopup(){
				this.form={
					avatar:'',
					avatarValue:'',
					name:'',
				}
			},
			chooseAvatar(){
				let that=this;
				uni.chooseImage({
				    count: 1, 
				    success: function (res) {
				        console.log(res);
						uni.getFileSystemManager().readFile({
							filePath: res.tempFilePaths[0],
							encoding: 'base64',
							success: r => {
								console.log(r);
								 that.form.avatar='data:image/png;base64,'+r.data;
								 that.form.avatarValue=r.data;
							},
							
						})
					}
				});
			},
			toNav(index) {
				console.log(index);
				if (globalData.register === 1) {
					if (index === 1) {
						uni.navigateTo({
							url: '../createSearchName/createSearchName'
						})
					} else if (index === 2) {
						uni.navigateTo({
							url: '../formwork/formwork'
						})
					} else if (index === 3) {
						uni.navigateTo({
							url: '../searchHistory/searchHistory'
						})
					}
				} else {
					uni.showToast({
						title: '请登陆后操作',
						icon: 'none'
					})
				}

			},
			toRegister() {
				let that = this;
				if(globalData.register===0 && globalData.err && (globalData.err_count)%3===0){
					uni.showModal({
						title:'提示',
						content:'账号违纪次数过多，请联系管理员解封'
					})
				}else{
					// #ifdef MP-WEIXIN
					wx.getUserProfile({
						desc: '毕业设计',
						success(info) {
							let userinfo = info.userInfo;
							uni.request({
								method: 'POST',
								url: globalData.baseUrl + '/user/register',
								data: {
									openid: globalData.openid,
									avatar: userinfo.avatarUrl,
									city: userinfo.city,
									country: userinfo.country,
									gender: userinfo.gender,
									nick_name: userinfo.nickName,
									province: userinfo.province
								},
								success(res) {
									if (res.data.errCode === 0) {
										uni.showToast({
											title: res.data.errMsg,
											icon: 'none'
										})
										globalData.register = 1;
										that.getUserInfo();
									}
								}
							})
						},
						fail(err) {
							console.log(err);
						}
					})
					// #endif
				}
				
			},
			getUser() {
				if (globalData.register == 1) {
					this.getUserInfo();
				}
			},
			updateInfo(){
				if(this.form.name===''){
					uni.showToast({
						title:'姓名不能为空',
						icon:'none'
					})
				}else{
					let that = this;
					uni.request({
						url: globalData.baseUrl + '/user/updateInfo',
						method: 'POST',
						data: {
							openid: globalData.openid,
							avatar:that.form.avatarValue,
							name:that.form.name,
						},
						success(res) {
							console.log(res);
							if (res.data.errCode === 0) {
								uni.showToast({
									title:res.data.errMsg,
									icon:'none'
								})
								that.userInfo.avatar=that.form.avatar;
								that.userInfo.name=that.form.name;
								that.show=false;
								
								/* that.getUserInfo(); */
							}
						}
					})
				}
			},
			toDetail() {
				this.form.avatar=this.userInfo.avatar
				this.form.name = this.userInfo.name
				this.show = true;
			},
			getUserInfo() {
				let that = this;
				uni.request({
					url: globalData.baseUrl + '/user/getUserInfo',
					method: 'POST',
					data: {
						openid: globalData.openid,
					},
					success(res) {
						if (res.data.errCode === 0) {
							that.userInfo = res.data.data;
							globalData.userInfo = that.userInfo;
						}
					}
				})
			}
		}
	}
</script>

<style lang="scss">
	button {
		//边框清掉
		border: 0;
		outline: none;
		margin: 0;
		//背景颜色清掉
		background-color: transparent;
	}

	button::after {
		border: none;
		background-color: none;
	}

	page {
		background: #f7f9fb;
		width: 100%;
		height: 100%;
	}

	.my-page {
		margin: 0 20rpx 0 20rpx;
	}

	.background {
		display: flex;
		align-items: center;
		width: 710rpx;
		height: 250rpx;
		background-color: #07d165;
		box-shadow: 1px 1px 8px #8ef6b1;
		color: white;
		border-radius: 10rpx;

		.myDetail {
			margin-left: 40rpx;
			display: flex;
			align-items: center;
		}
	}

	.menu {
		width: 710rpx;
		border-radius: 10rpx;
		transform: translateY(-70rpx);
		background-color: white;
		box-shadow: 1px 1px 4px #ebebeb;

		.header {
			color: white;
			display: flex;
			align-items: center;
			width: 100%;
			height: 80rpx;
			border-radius: 10rpx 10rpx 0 0;
			background-color: #3e4057;
		}

		.menu-item {
			.nav {
				display: flex;
				align-items: center;

			}

			margin:0 20rpx;
			height: 100rpx;
			border-bottom: 1px solid #dddddd;
			display: flex;
			align-items: center;
			justify-content: space-between;
		}

		.menu-item:last-child {
			border-bottom: none;
		}
	}
</style>
