<template>
	<view class="index-page">
		<view style="height: 20rpx;"></view>
		<u-search placeholder="搜索问卷" v-model="searchWord" @search="getSearchList" @custom="getSearchList"></u-search>
		<scroll-view  scroll-y class="scrollList" :style="{height:scrollHeight+'px'}" @scrolltolower="addPage">
			<view style="text-align: center;margin-top:20rpx;font-size: 36rpx;font-weight: 700;">问卷大厅</view>
			<view style="display: flex;flex-wrap: wrap;">
				<view class="scroll-item" v-for="(item,index) in searchList" :key="item.id" @click="toSearch(item.id)">
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
	</view>
</template>

<script>
	const globalData=getApp().globalData;
	export default {
		data() {
			return {
				searchWord: '',
				scrollHeight: 500,
				page:1,
				searchList:[],
				refresh:false,
			}
		},
		onLoad() {
			let that = this;
			uni.getSystemInfo({
				success(res) {
					that.scrollHeight = res.windowHeight * 0.85
				}
			})
		},
		onShow(){
			this.searchList=[];
			this.page=1;
			this.refresh=false;
			this.searchQuestion();
		},
		methods: {
			getSearchList(){
				console.log('点击');
				this.page=1;
				this.refresh=false;
				this.searchList=[];
				this.searchQuestion();
			},
			addPage(){
				if(!this.refresh){
					this.page+=1;
					this.searchQuestion();
				}else{
					uni.showToast({
						title:'到底啦',
						icon:'none'
					})
				}
			},
			toSearch(id){
				uni.navigateTo({
					url:'../search/search?id='+id
				})
			},
			searchQuestion(){
				let that=this;
				uni.request({
					method:'POST',
					url:globalData.baseUrl+'/search/searchQuestion',
					data:{
						page:that.page,
						name:that.searchWord
					},
					success(res) {
						if(res.data.errCode===0){
							console.log(res.data.data.length);
							if(res.data.data.length<6){
								that.refresh=true;
							}
							that.searchList=that.searchList.concat(res.data.data);
						}else{
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
					.image{
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
</style>
