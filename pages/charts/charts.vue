 <template>
	<view class="chart-page">
		<view style="height: 20rpx;"></view>
		<view style="font-weight: 700;font-size: 32rpx;text-align: center;margin-bottom: 20rpx;">图表统计</view>
		<uni-list>
		    <uni-list-item :title="item.name" link clickable  @click="toChart(item.id)" v-for="item in questionList" :key="item.id" v-if="item.question_type_id===1 ||item.question_type_id===2"></uni-list-item>
		</uni-list>
	</view>
</template>

<script>
	import uniList from "../../components/uni-list/uni-list.vue";
	import uniListItem from "../../components/uni-list-item/uni-list-item.vue";
	const glodbalData = getApp().globalData;
	export default {
		components:{
			uniList,
			uniListItem
		},
		data() {
			return {
				questionList: [],
			}
		},
		onLoad(options) {
			console.log(options);
			this.getQuestionBySearchId(options.id);
		},
		methods: {
			toChart(id){
				console.log(id);
				uni.navigateTo({
					url:'../showCharts/showCharts?id='+id
				})
			},
			getQuestionBySearchId(id) {
				let that = this;
				uni.request({
					method: 'POST',
					url: glodbalData.baseUrl + '/question/getQuestionBySearchId',
					data: {
						id,
					},
					success(res) {
						console.log(res);
						if (res.data.errCode === 0) {
							that.questionList = res.data.data;
						}
					}
				})
			},
		}
	}
</script>

<style lang="scss">
page {
		background: #fafafa;
		width: 100%;
		height: 100%;
	}

	.chart-page {
		margin: 0 20rpx 0 20rpx;
	}
</style> 
