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
					<text class="text-red">{{ items.ownerCheck == 1 ? '已确认' : '未确认'}}</text>
				</view>
				<view class="flex justify-between" style="margin-top: 20rpx;">
					<text class="text-black">租客确认情况</text>
					<text class="text-red">{{ items.isBack ? '已确认' : '未确认'}}</text>
				</view>
				<view class="flex justify-center"><button class="cu-btn round bg-blue" style="margin-left: 20rpx;" @click="confirmBack(items)">确认退还</button></view>
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
	methods: {
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
					title: '押金信息不存在'
				});
			}
		},
		getMyDeposit() {
			uni.request({
				url: http.BaseUrl + 'order/findDepositsOwner',
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
		confirmBack(items) {
			if (items.isBack === 0) {
				if (items.isBreak == 1) {
					uni.showToast({
						title: '租客违反合同，您不能退回押金',
						icon: 'none'
					});
				} else {
					if(items.isFinish === 1){
						uni.request({
							url: http.BaseUrl + 'order/ownerConfirm/' + items.did,
							method: 'PUT',
							header: {
								Authorization: 'bearer ' + _this.access_token
							},
							success: res => {
								_this.checkResponse(res);
								if (res.data.code === 200) {
									uni.showToast({
										title: '修改成功',
										icon: 'success'
									});
									_this.getMyDeposit();
								}
							}
						});
					}else{
						uni.showToast({
							title:"订单未完成，不能退回",
							icon:"none"
						})
					}
				}
			} else {
				uni.showToast({
					title: '您已退还',
					icon: 'none'
				});
			}
		}
	}
};
</script>

<style></style>
