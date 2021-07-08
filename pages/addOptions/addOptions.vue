<template>
	<view class="options-page">
		<view style="height: 20rpx;"></view>
		<view class="add-page">
			<u-form label-position="top" :model="form" ref="uForm">
				<u-form-item label="类型:" prop="name">
					<u-input v-model="form.type" type="select" border @click="show = true" />
					<u-select v-model="show" value-name="id" label-name="name" :list="typeList" @confirm="handleSelect">
					</u-select>
				</u-form-item>
				<u-form-item label="题目:" prop="desc">
					<u-input border v-model="form.title" />
				</u-form-item>
			</u-form>
			<view style="margin-top: 10rpx;">
				<u-form>
					<u-form-item :label="englishList[index]+'.'" v-for="(item,index) in optionsList" :key="index">
						<u-input v-model="item.value"></u-input>
						<u-icon name="trash" slot="right" size="50" @click="del(index)"></u-icon>
					</u-form-item>
				</u-form>
			</view>
		</view>
		<view class="btn">
			<view class="btn-options">
				<u-button type="primary" @click="addOptions" v-if="typeValue===2 || typeValue===1">添加选项</u-button>
			</view>
			<view class="btn-options">
				<u-button type="success" @click="submit">提交问题</u-button>
			</view>
		</view>
	</view>
</template>

<script>
	const globalData = getApp().globalData;
	export default {
		data() {
			return {
				typeList: [],
				englishList: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H'],
				show: false,
				form: {
					type: '',
					title: '',
				},
				typeValue: '',
				rules: {
					type: [{
						required: true,
						message: '请选择问题类型',
						trigger: ['change', 'blur'],
					}],
					title: [{
						required: true,
						message: '请输入问题题目',
						trigger: ['change', 'blur']
					}]
				},
				optionsList: [],
				id: 0,
			}
		},
		onReady() {
			this.$refs.uForm.setRules(this.rules);
		},
		onLoad(options) {
			this.id = options.id;
			this.getType();
		},
		methods: {
			submit() {
				let that = this;
				this.$refs.uForm.validate(valid => {
					if (valid) {
						if (that.typeValue === 1 || that.typeValue === 2) {
							that.optionsList.forEach(item => {
								if (item.value.trim() === '') {
									return uni.showToast({
										title: '选项请勿为空',
										icon: 'none'
									})
								}
							})
							let obj = {
								name: that.form.title,
								type: that.typeValue,
								id: that.id,
							}
							that.submitOptions(obj, that.optionsList);
						}else{
							let obj = {
								name: that.form.title,
								type: that.typeValue,
								id: that.id,
							}
							that.submitOptions(obj, that.optionsList);
						}
					} else {
						console.log('验证失败');
					}
				});
			},
			submitOptions(obj, options) {
				let that = this;
				options = JSON.stringify(options)
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/question/createQuestion',
					data: {
						name: obj.name,
						question_type_id: obj.type,
						search_name_id: obj.id,
						options,
					},
					success(res) {
						if (res.data.errCode === 0) {
							uni.navigateBack()
						} else {
							uni.showToast({
								title: res.data.errMsg,
								icon: 'none'
							})
						}
					}
				})
			},
			addOptions() {
				if (this.optionsList.length === 7) {
					uni.showToast({
						title: '无法继续添加',
						icon: 'none'
					})
				} else {
					let obj = {
						value: '',
						selected: 0,
					}
					this.optionsList.push(obj);
				}

			},
			del(index) {
				this.optionsList.splice(index, 1);
			},
			handleSelect(e) {
				console.log(e);
				this.form.type = e[0].label;
				this.typeValue = e[0].value;
				if(this.typeValue===3){
					this.optionsList=[];
				}
			},
			getType() {
				let that = this;
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/question/getType',
					success(res) {
						console.log(res);
						if (res.data.errCode === 0) {
							that.typeList = res.data.data;
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

	.options-page {
		margin: 0 20rpx 0 20rpx;
	}

	.add-page {
		box-shadow: 1px 1px 4px #ebebeb;
		background-color: white;
		border-radius: 10rpx;
		padding: 20rpx;
	}

	.btn {
		padding-top: 20rpx;
		width: 100%;
		display: flex;
		justify-content: space-between;

		.btn-options {
			width: 300rpx;
		}
	}
</style>
