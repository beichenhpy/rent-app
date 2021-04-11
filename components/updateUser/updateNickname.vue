<template>
	<view>
		<view class="cu-modal" :class="nickModal ? 'show' : ''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">修改昵称</view>
					<view class="action" @tap="updateNickName('close')"><text class="cuIcon-close text-red"></text></view>
				</view>
				<view class="padding-xl">
					<view class="cu-form-group margin-top" :class="updateOk ? '' : 'warn'">
						<view class="title">昵称</view>
						<input placeholder="输入修改的昵称" name="input" v-model="nickName"/>
					</view>
				</view>
				<view class="cu-bar bg-white">
					<view class="action margin-0 flex-sub text-green solid-left" @tap="updateNickName('close')">取消</view>
					<view class="action margin-0 flex-sub  solid-left" @tap="updateNickName('confirm')">确定</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
import http from '../../common/http.js';
export default {
	data() {
		return {
			nickName: '',
			updateOk:false
		};
	},
	props: {
		nickModal: {
			type: Boolean,
			default: false
		}
	},
	methods: {
		updateNickName(type) {
			if (type === 'close') {
				this.$emit('updateNickName', 'close');
			} else {
				if (this.nickName.length >= 20 || this.nickName.length == 0) {
					uni.showToast({
						title: '昵称不合法,修改失败',
						icon: 'none'
					});
					this.$emit('updateNickName', 'updateNo');
				} else {
					let that = this;
					let access_token = uni.getStorageSync('access_token');
					let userInfo = uni.getStorageSync('userInfo')
					uni.request({
						url:http.BaseUrl + 'user/findNickNameExist',
						method:"GET",
						header: {
							Authorization: 'bearer ' + access_token
						},
						data:{
							nickName:that.nickName
						},
						success: (res) => {
							if(res.data.code === 200){
								if(!res.data.data){
									that.updateOk = true;
									uni.request({
										url: http.BaseUrl + 'user/updateUser',
										method: 'PUT',
										data:{
											type:'nickName',
											profile:that.nickName
										},
										header: {
											Authorization: 'bearer ' + access_token
										},
										success: function(res) {
											if (res.data.code === 500) {
												uni.showToast({
													icon: 'none',
													title: '服务器开小差了'
												});
											} else if (res.data.code === 510) {
												uni.showToast({
													icon: 'none',
													title: res.data.data
												});
												setTimeout(function() {
													uni.redirectTo({
														url: '../login/reLogin?username=' + userInfo.username
													});
												}, 1000);
											} else if (res.data.code === 200) {
												that.$emit('updateNickName', 'updateOk');
											} else {
												uni.showToast({
													icon: 'none',
													title: '服务器异常'
												});
											}
										}
									});
								}else{
									that.updateOk = false;
									uni.showToast({
										title:"昵称已被占用",
										icon:"none"
									})
								}
							}
						}
					})
				}
			}
		}
	}
};
</script>

<style scoped>
	.warn{
		border-bottom: 2rpx solid red;
	}
</style>
