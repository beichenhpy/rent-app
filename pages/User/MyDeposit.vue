<template>
	<view>
		<view v-if="depositList.length === 0" class="flex justify-center" style="margin-top: 20rpx;">暂无数据</view>
		<view v-for="(items, index) in depositList" :key="index">
			<uni-card title="押金详情" :is-shadow="true" :extra="items.isBack === 1 ? '已退回' : '未退回'">
				<view class="flex justify-between">
					<text class="text-black">基本押金</text>
					<text class="text-red" style="margin-left: 20rpx;">{{ items.basicDeposit }}元</text>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<text class="text-black">钥匙押金</text>
					<text class="text-red">{{ items.keyDeposit }}元</text>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<text class="text-black">房东确认情况</text>
					<text class="text-red">{{ items.ownerCheck == 1 ? '已确认' : '未确认' }}</text>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<text class="text-black">租客确认情况</text>
					<text class="text-red">{{ items.isBack ? '已确认' : '未确认' }}</text>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<button class="cu-btn round bg-yellow" style="margin-left: 20rpx;" @click="back(items)">申请退还</button>
					<button class="cu-btn round bg-green" style="margin-left: 20rpx;" @click="renterConfirm(items)">确认退还</button>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<button class="cu-btn round bg-grey" style="margin-left: 20rpx;" @click="printDeposit(items)">打印押金单</button>
					<button class="cu-btn round bg-blue" style="margin-left: 20rpx;" @click="showDeposit(items)">查看押金单</button>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<button class="cu-btn round bg-red" style="margin-left: 20rpx;" @click="deleteDeposit(items)">删除押金信息</button>
				</view>
			</uni-card>
		</view>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
export default {
	data() {
		return {
			depositList: [],
			access_token: '',
			userInfo: {}
		};
	},
	onLoad() {
		_this = this;
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
	},
	onShow() {
		let ok = this.checkToken(this.access_token);
		if (ok) {
			this.getMyDeposit();
		}
	},
	components: {
		uniCard
	},
	onPullDownRefresh() {
		this.getMyDeposit();
		uni.stopPullDownRefresh();
	},
	methods: {
		checkOk(items) {
			if (items.isBreak == 1) {
				uni.showToast({
					title: '您违反合同约定，不能退还押金',
					icon: 'none'
				});
				return false;
			}
			if (items.isBack === 1) {
				uni.showToast({
					title: '您的押金已退回',
					icon: 'none'
				});
				return false;
			}
			return true;
		},
		checkToken(access_token) {
			if (access_token === '') {
				uni.showToast({
					title: '未登录',
					icon: 'none'
				});
				setTimeout(function() {
					uni.navigateTo({
						url: '../login/login'
					});
				}, 1000);
				return false;
			} else {
				return true;
			}
		},
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
		getMyDeposit() {
			uni.request({
				url: http.BaseUrl + 'order/findDeposits',
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.depositList = res.data.data;
					}
				}
			});
		},
		back(deposit) {
			let ok = this.checkOk(deposit);
			if (ok) {
				if (deposit.isFinish == 1) {
					uni.showLoading({
						title: '正在申请'
					});
					uni.request({
						url: http.BaseUrl + 'order/askBackDeposit/' + deposit.did,
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
				} else {
					uni.showToast({
						title: '订单未完成，不能退租',
						icon: 'none'
					});
				}
			}
		},
		printDeposit(items) {
			if (items.isBreak == 1) {
				uni.showToast({
					title: '您违反合同约定，不能打印',
					icon: 'none'
				});
			} else {
				if (items.isFrozen == 1) {
					uni.showLoading({
						title: '正在下载'
					});
					uni.request({
						url: http.BaseUrl + 'file/getDeposit',
						method: 'GET',
						data: {
							oid: items.oid
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
												uni.setStorageSync(items.did, res.savedFilePath);
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
		showDeposit(items) {
			let path = uni.getStorageSync(items.did);
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
		renterConfirm(items) {
			let ok = this.checkOk(items);
			if (ok) {
				if (items.isFinish == 0) {
					uni.showToast({
						title: '订单未完成，不能确认',
						icon: 'none'
					});
				} else {
					uni.request({
						url: http.BaseUrl + 'order/renterConfirm/' + items.did,
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
								_this.getMyDeposit();
							}
						}
					});
				}
			}
		},
		deleteDeposit(item) {
			if (item.isFinish == 0) {
				uni.showToast({
					title: '不能删除，订单尚未完成',
					icon: 'none'
				});
			} else {
				uni.showModal({
					title:"删除押金信息",
					content:"删除后押金信息消实且不可逆，是否确定?",
					success: (res) => {
						if(res.confirm){
							uni.showLoading({
								title:"请稍后"
							})
							uni.request({
								url: http.BaseUrl + 'order/deleteDeposit/' + item.did,
								method: 'DELETE',
								header: {
									Authorization: 'bearer ' + _this.access_token
								},
								success: res => {
									uni.hideLoading()
									_this.checkResponse(res);
									if (res.data.code == 200) {
										uni.showToast({
											title: '删除成功',
											icon: 'success'
										});
										_this.getMyDeposit();
									}
								}
							});
						}
					}
				})
			}
		}
	}
};
</script>

<style></style>
