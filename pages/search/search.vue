<template>
	<view class="search-page">
		<view style="height: 20rpx;"></view>

		<view class="search">
			<u-form label-position="top" :model="form" ref="uForm">
				<!-- <u-form-item label="海报:" >
					<u-upload ref="uUpload" max-count="1" width="325" height="250" :action="action" :auto-upload="false" ></u-upload>
				</u-form-item> -->
				<u-form-item label="名称:" prop="name">
					<u-input v-model="form.name" disabled border />
				</u-form-item>
				<u-form-item :border-bottom="false" label="简介:" prop="desc">
					<u-input type="textarea" border disabled v-model="form.desc" />
				</u-form-item>
			</u-form>
		</view>

		<view class="search">
			<view class="question-item" v-for="(item,index) in searchList" :key="item.id">
				<view class="title">
					{{item.name}}:
				</view>
				<view class="options-item">
					<view style="margin-left: 50rpx;" v-if="item.question_type_id===1" >
						<u-radio-group wrap v-model="item.select">
							<u-radio   v-for="(item2, index2) in item.children" :name="item2.id" :key="index2">
								{{item2.value}}
							</u-radio>
						</u-radio-group>
					</view>
					
					<view style="margin-left: 50rpx;" v-else-if="item.question_type_id===2" >
						<u-checkbox-group wrap >
							<u-checkbox   v-for="(item2, index2) in item.children" v-model="item2.select" :name="item2.id" :key="item2.id">
								{{item2.value}}
							</u-checkbox>
						</u-checkbox-group>
					</view>
					
					<view v-else-if="item.question_type_id===3">
						<view v-for="(item2,index2) in item.children" :key="item2.id">
							<u-input v-model="item2.value" border type="textarea"></u-input>
						</view>
					</view>
				</view>
			</view>
		</view>
		
		<u-button @click="submit" type="success">提交问卷</u-button>
	</view>
</template>

<script>
	const globalData = getApp().globalData;
	export default {
		data() {
			return {
				name_id: 0,
				searchList: [],
				form: {
					name: '',
					desc: ''
				}
			}
		},
		onLoad(options) {
			console.log(options);
			this.name_id = options.id;
			this.getSearchById();
		},

		methods: {
			submit(){
				console.log(this.searchList);
				let ids=[];
				let fillList=[];
				let list=this.searchList;
				list.forEach(item=>{
					if(item.question_type_id===1){
						if(item.select!==0){
							ids.push(item.select);
						}
					}else if(item.question_type_id===2){
						item.children.forEach(item2=>{
							if(item2.select===true){
								ids.push(item2.id)
							}
						})
					}else if(item.question_type_id===3){
						item.children.forEach(item2=>{
							fillList.push(item2)
						})
					}
				})
				console.log(ids);
				console.log(fillList);
				this.submitSearch(ids,fillList)
			},
			submitSearch(ids,fillList){
				let data={
					ids,
					fillList
				}
				if(globalData.openid){
					data.openid=globalData.openid,
					data.name_id=this.name_id
				}
				let that=this;
				uni.request({
					method:'POST',
					url:globalData.baseUrl+'/search/submitSearch',
					data,
					success(res) {
						uni.showToast({
							title:res.data.errMsg,
							icon:'none'
						})
						if(res.data.errCode===0){
							setTimeout(function(){
								uni.switchTab({
									url:'../index/index'
								})
							},800)
						}
					}
				})
			},
			getSearchById() {
				let that = this;
				uni.request({
					method: 'POST',
					url: globalData.baseUrl + '/search/getSearchById',
					data: {
						id: that.name_id
					},
					success(res) {
						console.log(res);
						/* that.searchList=res.data.data; */
						let tempList = res.data.data;
						that.form.name = tempList.name;
						that.form.desc = tempList.desc;
						let list=tempList.children;
						list.forEach(item=>{
							item.select = 0
							if(item.question_type_id===2){
								item.children.forEach(item2=>{
									item2.select=false
								})
							}
						})
						console.log(list);
						that.searchList=list;
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

	.search-page {
		margin: 0 20rpx 0 20rpx;
	}

	.search {
		padding: 20rpx;
		border-radius: 15rpx;
		background-color: white;
		margin-bottom: 30rpx;

		.title {
			padding: 20rpx;
			font-weight: 700;
		}
	}
</style>
