<template>
	<view class="create-page">
		<view class="create">
			<u-form label-position="top" :model="form" ref="uForm">
				<u-form-item label="海报:">
					<u-upload ref="uUpload" max-count="1" width="325" height="250" :action="action" :auto-upload="false"
						@on-remove="remove" @on-choose-complete="change"></u-upload>
				</u-form-item>
				<u-form-item label="名称:" prop="name">
					<u-input v-model="form.name" border />
				</u-form-item>
				<u-form-item :border-bottom="false" label="简介:" prop="desc">
					<u-input type="textarea" border v-model="form.desc" />
				</u-form-item>
			</u-form>
			<view style="height: 40rpx;"></view>
			<u-button type="success" @click="submit">提交问卷</u-button>
		</view>

	</view>
</template>

<script>
	const globalData = getApp().globalData;
	import dayjs from "dayjs"
	export default {
		data() {
			return {
				action: 'http://www.example.com/upload',
				form: {
					name: '',
					desc: '',
					poster:'',
				},
				rules: {
					name: [{
						required: true,
						message: '请输入问卷名称',
						trigger: ['change', 'blur'],
					}],
					desc: [{
						required: true,
						max: 200,
						message: '简介不能为空且大于200字',
						trigger: ['change', 'blur']
					}]
				},
				copy:'',
			}
		},
		onLoad(options) {
			console.log(options);
			if(options.copy){
				this.copy=options.copy;
			}
		},
		onReady() {
			this.$refs.uForm.setRules(this.rules);
		},
		methods: {
			remove() {
				console.log('移除');
			},
			change(lists) {
				console.log(lists);
				let that=this;
				uni.getFileSystemManager().readFile({
					filePath: lists[0].url,
					encoding: 'base64',
					success: r => {
						that.form.poster=r.data;
					},
					
				})
			},
			submit() {
				let that = this;
				this.$refs.uForm.validate(valid => {
					if (valid) {
						let time = dayjs()
						that.createSearch(that.form.name, time, that.form.desc, globalData.openid,that.form.poster);
					} else {
						console.log('验证失败');
					}
				});
			},
			createSearch(name, start_time, desc, openid,poster) {
				let that=this;
				console.log(poster);
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/search/create',
					data: {
						name,
						start_time,
						desc,
						openid,
						poster
					},
					success(res) {
						console.log(res);
						if (res.data.errCode === 0) {
							let id = res.data.data.name_id;
							/* uni.navigateTo({
								url: '../createQuestion/createQuestion?id=' + id+'&state=1'
							}) */
							if(parseInt(that.copy)===1){
								let options=uni.getStorageSync('options')
								uni.request({
									method:'POST',
									url:globalData.baseUrl+'/question/copyQuestion',
									data:{
										name_id:id,
										question:JSON.stringify(options),
									},
									success(res2){
										if(res2.data.errCode===0){
											uni.navigateTo({
												url: '../createQuestion/createQuestion?id=' + id+'&state=1'
											})
										}else{
											uni.showToast({
												title:res2.data.errMsg,
												icon:'none'
											})
										}
									}
								})
							}else{
								uni.navigateTo({
									url: '../createQuestion/createQuestion?id=' + id+'&state=1'
								})
							}
						} else {
							uni.showToast({
								title: res.data.errMsg,
								icon: 'none'
							})
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
	page {
		background: #fafafa;
		width: 100%;
		height: 100%;
	}

	.create-page {
		margin: 0 20rpx 0 20rpx;
	}


	.create {
		padding: 20rpx;
		border-radius: 15rpx;
		background-color: white;

	}
</style>
