<template>
	<view>
		<view v-if="deposit != null">
			<uni-card title="押金详情" :is-shadow="true" note="true" :extra="deposit.isBack === 1 ? '已退回' : '未退回'">
				<view class="flex justify-between">
					<text class="text-black">基本押金</text>
					<text class="text-red" style="margin-left: 20rpx;">{{ deposit.basicDeposit }}元</text>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<text class="text-black">钥匙押金</text>
					<text class="text-red">{{ deposit.keyDeposit }}元</text>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<text class="text-black">房东确认情况</text>
					<text class="text-red">{{ deposit.ownerCheck == 1 ? '已确认' : '未确认' }}</text>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<text class="text-black">租客确认情况</text>
					<text class="text-red">{{ deposit.isBack ? '已确认' : '未确认' }}</text>
				</view>
				<template v-slot:footer>
					<view class="flex justify-between">
						<button class="cu-btn round bg-yellow" style="margin-left: 20rpx;" @click="back">申请退还</button>
						<button class="cu-btn round bg-green" style="margin-left: 20rpx;" @click="renterConfirm">确认退还</button>
					</view>
					<view class="flex justify-between" style="margin-top: 20rpx;">
						<button class="cu-btn round bg-grey" style="margin-left: 20rpx;" @click="printDeposit">打印押金单</button>
						<button class="cu-btn round bg-blue" style="margin-left: 20rpx;" @click="showDeposit">查看押金单</button>
					</view>
				</template>
			</uni-card>
		</view>
		<view v-else><view class="flex justify-center">暂无数据</view></view>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
export default {
	data() {
		return {
			oid: '',
			isFinish: 0,
			access_token: '',
			userInfo: {},
			deposit: {}
		};
	},
	onLoad(e) {
		_this = this;
		this.oid = e.oid;
		this.isFinish = e.isFinish;
	},
	onShow() {
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
		this.getDeposit();
	},
	onPullDownRefresh() {
		this.getDeposit();
		uni.stopPullDownRefresh()
	},
	components: {
		uniCard
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
		getDeposit() {
			uni.request({
				url: http.BaseUrl + 'order/findDeposit/' + _this.oid,
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.deposit = res.data.data;
					}
				}
			});
		},
		checkOk() {
			if (_this.deposit.isBreak == 1) {
				uni.showToast({
					title: '您违反合同约定，不能退还押金',
					icon: 'none'
				});
				return false;
			}
			if (this.deposit.isBack === 1) {
				uni.showToast({
					title: '您的押金已退回',
					icon: 'none'
				});
				return false;
			}
			if (this.isFinish == 0) {
				uni.showToast({
					title: '订单未完成，请退租后再申请',
					icon: 'none'
				});
				return false;
			}
			return true;
		},
		back() {
			let ok = this.checkOk();
			if (ok) {
				uni.showLoading({
					title: '正在申请'
				});
				uni.request({
					url: http.BaseUrl + 'order/askBackDeposit/' + _this.deposit.did,
					method: 'POST',
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					complete: res => {
						uni.hideLoading();
						_this.checkResponse(res);
						if (res.data.code === 200) {
							uni.showToast({
								title: '申请成功',
								icon: 'success'
							});
						}
					}
				});
			}
		},
		printDeposit() {
			if (_this.deposit.isBreak == 1) {
				uni.showToast({
					title: '您违反合同约定，无法打印',
					icon: 'none'
				});
				return;
			} else {
				if (this.isFrozen == 1) {
					uni.showLoading({
						title: '正在下载'
					});
					uni.request({
						url: http.BaseUrl + 'file/getDeposit',
						method: 'GET',
						data: {
							oid: _this.oid
						},
						complete: res => {
							uni.hideLoading();
							_this.checkResponse(res);
							if (res.data.code === 200) {
								uni.downloadFile({
									url: res.data.data,
									success: res => {
										uni.saveFile({
											tempFilePath: res.tempFilePath,
											success: res => {
												uni.setStorageSync(_this.deposit.did, res.savedFilePath);
												uni.showToast({
													title: '保存路径为:' + res.savedFilePath,
													icon: 'none'
												});
											}
										});
									}
								});
							}
						}
					});
				} else {
					uni.showToast({
						title: '订单未完成不能打印',
						icon: 'none'
					});
				}
			}
		},
		showDeposit() {
			let path = uni.getStorageSync(_this.deposit.did);
			if (path.length != 0) {
				uni.openDocument({
					filePath: path
				});
			} else {
				uni.showToast({
					title: '请重新下载',
					icon: 'none'
				});
			}
		},
		renterConfirm() {
			let ok = this.checkOk();
			if (ok) {
				uni.request({
					url: http.BaseUrl + 'order/renterConfirm/' + _this.deposit.did,
					method: 'PUT',
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					success: res => {
						_this.checkResponse(res);
						if (res.data.code == 200) {
							uni.showToast({
								title: '确认成功',
								icon: 'success'
							});
							_this.getDeposit();
						}
					}
				});
			}
		}
	}
};
</script>

<style></style>
