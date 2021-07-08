<template>
	<view class="statistic-page">
		<view style="height: 20rpx;"></view>
		<view class="table">
			<view style="font-weight: 700;font-size: 32rpx;text-align: center;">表格统计</view>
			<uni-collapse accordion="true">
				<uni-collapse-item :title="item.name" v-for="item in questionList" :key="item.id" v-if="item.question_type_id===1||item.question_type_id===2">
					<view>
						<u-table>
							<u-tr>
								<u-th>题目</u-th>
								<u-th>选择人数</u-th>
							</u-tr>
							<u-tr v-for="item2 in item.children" :key="item2.id">
								<u-td>{{item2.value}}</u-td>
								<u-td>{{item2.selected}}</u-td>
							</u-tr>
						</u-table>
					</view>
				</uni-collapse-item>
				
				<uni-collapse-item :title="item.name" v-for="item in questionList" :key="item.id" v-if="item.question_type_id===3">
					<view>
						<u-table>
							<u-tr>
								<u-th>填写情况</u-th>
							</u-tr>
							<u-tr v-for="item2 in item.children[0].children" :key="item2.id">
								<u-td>{{item2.value}}</u-td>
							</u-tr>
						</u-table>
					</view>
				</uni-collapse-item>
				
				
			</uni-collapse>
		</view>
	</view>
</template>

<script>
	import uniCollapse from "../../components/uni-collapse/uni-collapse.vue"
	import uniCollapseItem from "../../components/uni-collapse-item/uni-collapse-item.vue"
	const glodbalData = getApp().globalData;
	export default {
		components: {
			uniCollapse,
			uniCollapseItem
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
			getQuestionBySearchId(id) {
				let that = this;
				uni.request({
					method: 'POST',
					url: glodbalData.baseUrl + '/question/getQuestionBySearchId',
					data: {
						id,
					},
					success(res) {
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

	.statistic-page {
		margin: 0 20rpx 0 20rpx;
	}

	.table {
		background-color: white;
		padding: 20rpx;
		box-shadow: 1px 0px 1px #C0C0C0;
		border-radius: 10rpx;
	}
</style>
