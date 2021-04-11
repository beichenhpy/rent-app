<template>
	<view>
		<view>
			<checkbox-group class="block" @change="CheckboxChange">
				<view class="cu-form-group margin-top" v-for="(item, index) in items" :key="index">
					<view class="title">{{ item.name }}</view>
					<checkbox :class="item.checked ? 'checked' : ''" :checked="item.checked" :value="item.value"></checkbox>
				</view>
			</checkbox-group>
		</view>
		<view class="flex justify-center" style="margin-top: 50rpx;"><button class="cu-btn bg-green" @click="confirmUpdate">确认修改</button></view>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
export default {
	data() {
		return {
			houseInfo: {},
			access_token: '',
			userInfo: {},
			isRented: '',
			isOnRent: '',
			items: [
				{
					value: 'bed',
					name: '床',
					checked: false
				},
				{
					value: 'wash',
					name: '洗衣机',
					checked: false
				},
				{
					value: 'air',
					name: '空调',
					checked: false
				},
				{
					value: 'warm',
					name: '淋浴',
					checked: false
				},
				{
					value: 'robe',
					name: '衣柜',
					checked: false
				},
				{
					value: 'fridge',
					name: '冰箱',
					checked: false
				}
			]
		};
	},
	onLoad(e) {
		_this = this;
		this.houseInfo.hid = e.hid;
		this.houseInfo.houseInfoId = e.houseInfoId;
		this.isRented = e.isRented;
		this.isOnRent = e.isOnRent;
	},
	onShow() {
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
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
					uni.navigateTo({
						url: '../login/reLogin?username=' + _this.userInfo.username
					});
				}, 1000);
			}
			if (res.data.code === 400 || res.data.status === 500) {
				uni.showToast({
					icon: 'none',
					title: '服务器开小差了'
				});
			}
		},
		CheckboxChange(e) {
			var items = this.items,
				values = e.detail.value;
			for (var i = 0, lenI = items.length; i < lenI; ++i) {
				const item = items[i];
				if (values.includes(item.value)) {
					this.$set(item, 'checked', true);
				} else {
					this.$set(item, 'checked', false);
				}
			}
			if (e.detail.value.includes('bed')) {
				this.houseInfo.isBed = 1;
			} else {
				this.houseInfo.isBed = 0;
			}
			if (e.detail.value.includes('wash')) {
				this.houseInfo.isWash = 1;
			} else {
				this.houseInfo.isWash = 0;
			}
			if (e.detail.value.includes('warm')) {
				this.houseInfo.isWarm = 1;
			} else {
				this.houseInfo.isWarm = 0;
			}
			if (e.detail.value.includes('robe')) {
				this.houseInfo.isRobe = 1;
			} else {
				this.houseInfo.isRobe = 0;
			}
			if (e.detail.value.includes('fridge')) {
				this.houseInfo.isFridge = 1;
			} else {
				this.houseInfo.isFridge = 0;
			}
			if (e.detail.value.includes('air')) {
				this.houseInfo.isAir = 1;
			} else {
				this.houseInfo.isAir = 0;
			}
		},
		confirmUpdate() {
			uni.showLoading({
				title: '正在修改'
			});
			uni.request({
				url: http.BaseUrl + 'house/updateHouseInfo',
				data: _this.houseInfo,
				method: 'PUT',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				complete: function(res) {
					uni.hideLoading();
					_this.checkResponse(res);
					if (res.data.code === 200) {
						uni.showToast({
							title: '修改成功',
							icon: 'success'
						});
						uni.navigateBack({
							delta: 1
						});
					}
				}
			});
		}
	}
};
</script>

<style scoped></style>
