<template>
	<view class="history-page">
		<view style="height: 20rpx;"></view>
		<uni-list>
			<uni-list-item  v-for="(item,index) in formWorkList" :key="item.id" @click="toSearch(item.id)">
				<view slot="header">
					<image style="height: 150rpx;width: 150rpx;" :src="item.poster"></image>
				</view>
				<view  slot="body">
					<view style="width:330rpx;margin: 0 20rpx;height: 100%;display: flex;flex-direction: column;">
						<view style="font-size: 32rpx;font-weight: 700;">
							{{item.name}}
						</view>
						<view style="font-size: 26rpx;color: #a5a5a5;">
							{{item.desc}}
						</view>
					</view>
				</view>
				<template slot="footer" style="width: 100%;display: flex;align-items: flex-end;justify-content: flex-end;">
					<view >
						<u-button size="mini" shape="circle"  type="success" @click="toSearch(item.id)">查看模板</u-button>
					</view>
				</template>
				
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
				formWorkList:[],
			}
		},
		onLoad(){
			this.getSearchList();
		},
		methods: {
			toSearch(id){
				console.log('id');
				uni.navigateTo({
					url:'../createQuestion/createQuestion?id='+id
				})
			},
			getSearchList(){
				let that=this;
				uni.request({
					method:"POST",
					url:globalData.baseUrl+'/search/getformWork',
					data:{
						openid:'o2P8s5M8BWPXM1FACqTLfglQXskM'
					},
					success(res) {
						console.log(res);
						if(res.data.errCode===0){
							that.formWorkList=res.data.data;
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
