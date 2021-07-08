<template>
	<view class="history-page">
		<view style="height: 20rpx;"></view>
		<uni-list>
			<uni-list-item link v-for="(item,index) in historyList" :key="item.id" @click="toSearch(item.parent.id)">
				<view slot="header">
					<image style="height: 150rpx;width: 150rpx;" :src="item.parent.poster"></image>
				</view>
				<view  slot="body">
					<view style="margin: 0 20rpx;height: 100%;display: flex;flex-direction: column;justify-content:space-between">
						<view style="font-size: 32rpx;font-weight: 700;">
							{{item.parent.name}}
						</view>
						<view style="font-size: 26rpx;color: #a5a5a5;">
							{{item.created_time}}
						</view>
					</view>
				</view>
			</uni-list-item>
		</uni-list>
	</view>
</template>

<script>
	import dayjs from "dayjs"
	import uniList from "../../components/uni-list/uni-list.vue";
	import uniListItem from "../../components/uni-list-item/uni-list-item.vue"
	const globalData=getApp().globalData;
	export default {
		components:{
			uniList,
			uniListItem
		},
		data() {
			return {
				historyList:[],
			}
		},
		onLoad(){
			this.getSearchList();
		},
		methods: {
			toSearch(id){
				uni.navigateTo({
					url:'../search/search?id='+id
				})
			},
			getSearchList(){
				let that=this;
				uni.request({
					method:"POST",
					url:globalData.baseUrl+'/user/getUserHistory',
					data:{
						openid:globalData.openid
					},
					success(res) {
						console.log(res);
						if(res.data.errCode===0){
							that.historyList=res.data.data;
							that.historyList.forEach(item=>{
								item.created_time=dayjs(item.created_time).format('YYYY-MM-DD')
							})
						}else {
							uni.showToast({
								title:res.data.errMsg,
								icon:'none'
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

	.history-page {
		margin: 0 20rpx 0 20rpx;
	}

</style>
