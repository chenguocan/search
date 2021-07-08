<template>
	<view class="create-page">
		<view style="height: 20rpx;"></view>
		<view style="font-size: 48rpx; font-weight: 700;margin: 20rpx 0;display: flex;justify-content: space-between;align-items: center;">
			<view>题目:</view>
			<view style="display: flex;flex-direction: column;align-items: center;justify-content: center;">
				<view >
					<i  class="iconfont icon-fuzhi" style="font-size: 48rpx;" @click="toCopy"></i>
				</view>
			</view>
		</view>
		<view class="question-item" v-for="(item,index) in questionList" :key="item.id">
			<view class="name">
				<text style="font-size: 36rpx;font-weight: 700;margin-right: 20rpx;">{{index+1}}.</text>
				<text style="font-size: 32rpx;font-weight: 700;">{{item.name}}</text>
				<text>[{{typeList[item.question_type_id-1]}}]</text>
			</view>
			<view style="margin:15rpx 0 15rpx 40rpx;" v-for="(item2,index2) in item.children" :key="item.id" v-if="item.question_type_id!==3">
				<text style="font-weight: 700;margin-right: 10rpx;">{{englishList[index2]}}.</text><text>{{item2.value}}</text>
			</view>
			<view class="btn-list">
				<view class="btn-item" @click="edit(item.id)" v-if="parseInt(state)===1  && state">
					<view class="iconfont icon-icon_edit"></view>
					<view class="options">编辑</view>
				</view>
				<view class="btn-item" @click="deleteOptions(item.id)" v-if="parseInt(state)===1  && state">
					<view class="iconfont icon-shanchu"></view>
					<view class="options" >删除</view>
				</view>
			</view>
		</view>
		<view class="btn">
			<view class="btn-options">
				<u-button type="primary" @click="addOptions" v-if="parseInt(state)===1">添加题目</u-button>
			</view>
			<view class="btn-options">
				<u-button type="success" @click="submit" v-if="parseInt(state)===1">提交问卷</u-button>
			</view>
		</view>
	</view>
</template>

<script>
	const globalData = getApp().globalData;
	export default {
		data() {
			return {
				name_id: 0,
				questionList:[],
				englishList:['A','B','C','D','E','F','G','H'],
				typeList:['单选','多选','填空'],
				state:0,
			}
		},
		onLoad(options) {
			console.log(options)
			if(options.state){
				this.state=options.state;
			}
			this.name_id = options.id;
		},
		onShow(){
			this.getQuestionList();
		},
		methods: {
			toCopy(){
				let options=[];
				let storyOptions=uni.getStorageSync('options');
				if(storyOptions){
					uni.clearStorageSync('options')
				}
				let list=this.questionList;
				list.forEach(item=>{
					let obj={};
					obj.name=item.name;
					obj.question_type_id=item.question_type_id;
					let tempList=[];
					item.children.forEach(item2=>{
						let temobj={};
						temobj.value=item2.value;
						temobj.selected=0
						tempList.push(temobj)
					})
					obj.children=tempList;
					options.push(obj)
				})
				uni.setStorageSync('options',options);
				uni.navigateTo({
					url:'../createSearchName/createSearchName?copy=1'
				})
			},
			edit(id){
				let that=this;
				uni.navigateTo({
					url:'../editOptions/editOptions?name_id='+that.name_id+'&question_id='+id
				})
			},
			deleteOptions(id){
				let that=this;
				 uni.request({
					url:globalData.baseUrl+'/question/deleteQuestion',
					method:'POST',
					data:{
						id,
					},
					success(res) {
						if(res.data.errCode===0){
							that.getQuestionList();
						}
						uni.showToast({
							title:res.data.errMsg,
							icon:'none'
						})
					}
				}) 
			},
			addOptions(){
				uni.navigateTo({
					url:'../addOptions/addOptions?id='+this.name_id
				})
			},
			submit(){
				let that=this;
				if(this.questionList.length!==0){
					uni.request({
						method:'POST',
						url:globalData.baseUrl+'/search/submitVerify',
						data:{
							id:that.name_id,
						},
						success(res){
							uni.showToast({
								title:res.data.errMsg,
								icon:'none'
							})
							if(res.data.errCode===0){
								uni.switchTab({
									url:'../index/index'
								})
							}
						}
					})
				}else{
					uni.showToast({
						title:'该问卷没有题目，无法提交',
						icon:'none'
					})
				}
			},
			getQuestionList() {
				let that = this;
				uni.request({
					url: globalData.baseUrl + '/question/getQuestionList',
					method: 'POST',
					data: {
						id: that.name_id,
					},
					success(res) {
						that.questionList=res.data.data;
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
	.question-item{
		box-shadow: 1px 1px 4px #ebebeb;
		background-color: white;
		border-radius: 10rpx;
		padding: 20rpx;
		margin-bottom: 20rpx;
		.btn-list{
			display: flex;
			justify-content: flex-end;
			.btn-item{
				display: flex;
				flex-direction: column;
				justify-content: center;
				align-items: center;
				padding: 0 20rpx 0 20rpx;
				font-size: 24rpx;
				.iconfont{
					font-size: 48rpx;
				}
			}
		}
	}
	.btn{
		padding-top: 20rpx;
		width: 100%;
		display: flex;
		justify-content: space-between;
		.btn-options{
			width: 300rpx;
		}
	}
</style>
