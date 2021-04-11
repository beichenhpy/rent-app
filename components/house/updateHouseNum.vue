<template>
	<view class="cu-modal" :class="Modal ? 'show' : ''">
		<view class="cu-dialog">
			<view class="cu-bar bg-white justify-end"><view class="content">修改房产证号</view></view>
			<view class="padding-xl">
				<view class="cu-form-group margin-top">
					<view class="title">房产证号</view>
					<input placeholder="输入修改的房产证号" type="number" maxlength="8" v-model="houseCardNum" />
				</view>
			</view>
			<view class="cu-bar bg-white">
				<view class="action margin-0 flex-sub text-green solid-left" @tap="updateHouseNum('close')">取消</view>
				<view class="action margin-0 flex-sub  solid-left" @tap="updateHouseNum('confirm')">确定</view>
			</view>
		</view>
	</view>
</template>

<script>
import http from '../../common/http.js';
export default {
	data() {
		return {
			houseCardNum: ''
		};
	},
	props: {
		Modal: {
			type: Boolean,
			default: false
		},
		hid: {
			type: String
		}
	},
	methods: {
		checkResponse(res) {
			if (res.data.code === 500) {
				uni.showToast({
					icon: 'none',
					title: '服务器开小差了'
				});
			}
			if (res.data.code === 510) {
				uni.showToast({
					icon: 'none',
					title: res.data.data
				});
				setTimeout(function() {
					let userInfo = uni.getStorageSync('userInfo');
					uni.navigateTo({
						url: '../login/reLogin?username=' + userInfo.username
					});
				}, 1000);
			}
			if (res.data.code === 400) {
				uni.showToast({
					icon: 'none',
					title: res.data.data
				});
			}
		},
		updateHouseNum(type) {
			if (type === 'close') {
				this.$emit('updateHouseNum', 'close');
			} else {
				if (this.houseCardNum.length != 8) {
					uni.showToast({
						title: '不合法,应该为8位数字',
						icon: 'none'
					});
					this.$emit('updateHouseNum', 'updateNo');
				} else {
					let that = this;
					let access_token = uni.getStorageSync('access_token');
					
					uni.request({
						url: http.BaseUrl + 'house/updateHouseCardNum',
						method: 'PUT',
						data: {
							houseCardNum: that.houseCardNum,
							hid: that.hid
						},
						header: {
							Authorization: 'bearer ' + access_token
						},
						success: function(res) {
							that.checkResponse(res);
							if (res.data.code === 200) {
								that.$emit('updateHouseNum', 'updateOk');
							}
						}
					});
				}
			}
		}
	}
};
</script>

<style></style>
