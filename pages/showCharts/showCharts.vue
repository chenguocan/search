<template>
	<view>
		<view style="margin: 20rpx;border-bottom: 1px solid #CCCCCC;">
			<view style="display: flex;align-items: flex-end;">
				<text style="font-weight: 700;font-size: 36rpx;">{{search.name}}</text>
			</view>
			<view style="display:flex;justify-content: flex-end;">{{'['+typeList[search.question_type_id-1]+']'}}</view>
		</view>
		<view>
			<view class="title">柱状图</view>
			<view class="charts-box">
				<qiun-data-charts type="column" :chartData="chartData" />
			</view>
		</view>
		<view>
			<view class="title">饼状图</view>
			<view class="charts-box">
				<qiun-data-charts type="pie" :chartData="chartData2" />
			</view>
		</view>
		<view>
			<view class="title">漏斗图</view>
			<view class="charts-box">
				<qiun-data-charts type="funnel" :chartData="chartData2" />
			</view>
		</view>

	</view>
</template>

<script>
	
	const glodbalData = getApp().globalData;
	export default {
	
		data() {
			return {
				chartData: {
				},
				chartData2: {},
				currentId: 0,
				search:[],
				typeList:['单选','多选','填空']
			}
		},
		onLoad(options) {
			this.currentId = options.id;
			console.log(options);
			this.getQuestionById();
		},
		methods: {
			getQuestionById() {
				let that = this;
				uni.request({
					method: 'POST',
					url: glodbalData.baseUrl + '/question/getQuestionById',
					data: {
						id: that.currentId
					},
					success(res) {
						console.log(res)
						if (res.data.errCode === 0) {
							that.search=res.data.data;
							let columncategories = [];
							let columnData=[]
							let pieData = [];
							let tempData = res.data.data;
							tempData.children.forEach(item => {
								let obj = {}
								columncategories.push(item.value);
								obj = {
									name: item.value,
									value: item.selected
								}
								pieData.push(obj);
								columnData.push(item.selected)
							})
							that.chartData = {
									categories: columncategories,
									series: [{
										name: '选择人数',
										data: columnData
									}]
								},
								that.chartData2 = {
									"series": [{
										"data": pieData
									}]
								}
							console.log(that.chartData2);
						}

					},
					fail(err) {
						console.log(err);
					}
				})
			}
		}
	}
</script>

<style lang="scss">
	.charts-box {
		width: 750rpx;
		height: 400rpx;
		margin-bottom: 50rpx;
	}

	.title {
		font-weight: 700;
		font-size: 32rpx;
		text-align: center;
		margin-top: 20rpx;
	}
</style>
