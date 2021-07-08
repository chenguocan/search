<template>
	<view class="index-page">
		<view style="height: 20rpx;"></view>
		<u-tabs :bar-style="{backgroundColor:'#07d165'}" :active-item-style="{color:'#07d165'}" :list="list"
			:is-scroll="false" :current="current" @change="change"></u-tabs>
		<scroll-view scroll-y class="scrollList" :style="{height:scrollHeight+'px'}">
			<uni-notice-bar speed="50" scrollable="true" showIcon="true" single="true" :text="noticeText">
			</uni-notice-bar>
			<view style="display: flex;flex-wrap: wrap;">
				<view class="scroll-item" v-for="(item,index) in searchList" :key="item.id"
					@click="showOptions(item.id,item.state,item.public,item.err)">
					<view class="search-detail">
						<view class="search-img">
							<image class="image" :src="item.poster"></image>
						</view>
						<view class="search-name">
							<view>{{item.name}}</view>
						</view>
					</view>
				</view>
			</view>
		</scroll-view>

		<u-popup v-model="show" mode="bottom" border-radius="15" length="40%">
			<view class="popup">
				<view style="height: 50rpx;"></view>
				<view class="options-list">

					<view class="options-item">
						<view class="iconfont icon-search" @click="search"></view>
						<view>查看</view>
					</view>

					<view class="options-item" v-if="currentState===0" @click="vertify">
						<view class="iconfont icon-wenjuanzhongxin"></view>
						<view>审核</view>
					</view>

					<view class="options-item" v-if="currentState===3 && currentPublic===0" @click="publicSearch(1)">
						<view class="iconfont icon-fabu"></view>
						<view>发布</view>
					</view>

					<view class="options-item" v-else-if="currentState===3 && currentPublic===1"
						@click='publicSearch(0)'>
						<view class="iconfont icon-fabu"></view>
						<view>取消发布</view>
					</view>

					<view class="options-item" v-if="currentState===3 && currentPublic===1" @click="showQrCode">
						<view class="iconfont icon-erweima"></view>
						<view>二维码</view>
					</view>

					<view class="options-item" v-if="currentState===3" @click="statistics(1)">
						<view class="iconfont icon-tongji"></view>
						<view>表格统计</view>
					</view>

					<view class="options-item" v-if="currentState===3" @click="statistics(2)">
						<view class="iconfont icon-tubiao"></view>
						<view>图表统计</view>
					</view>

					<view class="options-item" v-if="currentState===4" @click="errShow=true">
						<view class="iconfont icon-info"></view>
						<view>错误</view>
					</view>

					<view class="options-item" @click="deleteSearch">
						<view class="iconfont icon-shanchu1"></view>
						<view>删除</view>
					</view>
				</view>
			</view>
		</u-popup>

		<u-popup v-model="qrShow" closeable mode="center" border-radius="15" width="600rpx" height="60%">
			<view style="width: 100%;height: 100%;display: flex;align-items: center;justify-content: center;">
				<view style="width: 430rpx;height: 430rpx;">
					<image style="width: 430rpx;height: 430rpx;" :src="currentImage"></image>
				</view>
			</view>
		</u-popup>

		<u-modal v-model="errShow" :content="content"></u-modal>
	</view>
</template>

<script>
	import uniNoticeBar from "../../components/uni-notice-bar/uni-notice-bar.vue";
	const globalData = getApp().globalData;
	export default {
		components: {
			uniNoticeBar
		},
		data() {
			return {
				noticeText: '',
				qrShow: false,
				show: false,
				errShow: false,
				content: '',
				searchWord: '',
				scrollHeight: 500,
				page: 0,
				searchList: [],
				list: [{
					name: '待提交'
				}, {
					name: '待审核'
				}, {
					name: '已审核'
				}, {
					name: '未通过'
				}],
				current: 0,
				currentId: 0,
				currentState: 0,
				currentPublic: 0,
				currentImage: '',
			}
		},
		onLoad() {
			let that = this;
			uni.getSystemInfo({
				success(res) {
					that.scrollHeight = res.windowHeight * 0.85
				}
			})
			this.noticeText = '创建问卷需要提交方可进入审核，若未提交，无法进入审核状态'

		},
		onShow() {
			this.show=false;
			this.qrShow=false;
			if (globalData.register === 1) {
				this.getSearchList();
			}
		},
		methods: {
			showQrCode() {
				let that = this;
				uni.showLoading({
					title: '二维码加载中',
				})
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/qrCode',
					data: {
						id: that.currentId
					},
					success(res) {
						if (res.data.errCode === 0) {
							that.currentImage = res.data.data;
							that.show = false;
							that.qrShow = true;
						} else {
							uni.showToast({
								title: res.data.errMsg,
								icon: 'none'
							})
						}
					},
					complete() {
						uni.hideLoading();
					}
				})

			},
			statistics(index) {
				let that = this;
				if (index === 1) {
					uni.navigateTo({
						url: '../statistics/statistics?id=' + that.currentId
					})
				} else if (index === 2) {
					uni.navigateTo({
						url: '../charts/charts?id=' + that.currentId
					})
				}
			},
			publicSearch(state) {
				let that = this;
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/search/publicSearch',
					data: {
						id: that.currentId,
						publicState: state,
					},
					success(res) {
						uni.showToast({
							title: res.data.errMsg,
							icon: 'none'
						})
						if (res.data.errCode === 0) {
							that.show = false;
							that.getSearchList();
						}
					}
				})
			},
			deleteSearch() {
				let that = this;
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/search/deleteSearch',
					data: {
						id: that.currentId
					},
					success(res) {
						uni.showToast({
							title: res.data.errMsg,
							icon: 'none'
						})
						if (res.data.errCode === 0) {
							that.show = false;
							that.getSearchList();
						}
					}
				})
			},
			vertify() {
				let that = this;
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/search/submitVerify',
					data: {
						id: that.currentId
					},
					success(res) {
						uni.showToast({
							title: res.data.errMsg,
							icon: 'none'
						})
						that.getSearchList();
						if (res.data.errCode === 0) {
							that.show = false
						}
					}
				})
			},
			search() {
				let that = this;
				uni.navigateTo({
					url: '../createQuestion/createQuestion?id=' + that.currentId + '&state=' + that.currentState,
					success() {
						that.show = false
					}
				})
			},
			showOptions(id, state, public_state, err) {
				this.show = true
				this.currentId = id;
				this.currentState = state;
				this.currentPublic = public_state;
				this.content = err
			},
			change(index) {
				this.current = index;
				if (parseInt(globalData.register) === 1) {
					if (this.current === 0) {
						this.noticeText = '创建问卷需要提交方可进入审核，若未提交，无法进入审核状态'
					}else if(this.current===1){
						this.noticeText = '等待管理员审核问卷'
					} else if (this.current === 2) {
						this.noticeText = '问卷审核完成，需要点击相应问卷进行发布'
					} else if (this.current === 3) {
						this.noticeText = '点击相应的问卷可以查看错误信息,请修改完成后提交审核'
					}
					this.getSearchList();
				}
			},
			getSearchList() {
				let that = this;
				let state;
				if (that.current == 0) {
					state = [1]
				} else if (that.current === 1) {
					state = [2]
				} else if (that.current == 2) {
					state = [3]
				} else if (that.current === 3) {
					state = [4]
				}
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/search/getSearchList',
					data: {
						page: that.page,
						openid: globalData.openid,
						state
					},
					success(res) {
						if (res.data.errCode === 0) {
							that.searchList = res.data.data;
						} else {
							uni.showToast({
								title: res.data.errMsg,
								icon: 'none'
							})
						}
					}

				})
			}
		}
	}
</script>

<style lang="scss">
	page {
		background: #fafafa;
		width: 100%;
		height: 100%;
	}

	.popup {
		margin: 0 50rpx;
	}

	.index-page {
		margin: 0 20rpx 0 20rpx;
	}

	.scrollList {
		background-color: #f2f2f2;
		border-radius: 15rpx;
		margin-top: 20rpx;
		box-shadow: 1px 1px 4px #ebebeb;

		.scroll-item {
			width: 325rpx;
			height: 350rpx;
			margin: 10rpx 0 10rpx 20rpx;
			background-color: white;
			border-radius: 10rpx;
			padding: 10rpx;
			box-shadow: 1px 1px 4px #ebebeb;

			.search-detail {
				width: 100%;
				height: 100%;

				.search-img {
					height: 250rpx;
					border-radius: 10rpx;

					.image {
						width: 100%;
						height: 250rpx;
						border-radius: 10rpx;
					}
				}

				.search-name {
					display: -webkit-box;

					overflow: hidden;

					text-overflow: ellipsis;

					word-wrap: break-word;

					white-space: normal !important;

					-webkit-line-clamp: 2;

					-webkit-box-orient: vertical;
				}
			}
		}
	}

	.options-list {
		display: flex;
		flex-wrap: wrap;
		align-items: center;

		.options-item {
			width: 150rpx;
			margin-top: 30rpx;
			margin-left: 10rpx;
			margin-bottom: 40rpx;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;

			.iconfont {
				font-size: 60rpx;
			}
		}
	}
</style>
