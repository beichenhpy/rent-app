<template>
	<view style="margin-bottom: 20%;">
		<view v-if="rentList.length === 0" class="flex justify-center" style="margin-top: 20rpx;">暂无数据</view>
		<view v-for="(items, index) in rentList" :key="index">
			<uni-card :title="items.title" :is-shadow="true" mode="basic">
				<view class="flex justify-between">
					<text class="weight">发布状态:</text>
					<text v-if="items.isOnRent == 1" class="left text-green">已发布</text>
					<text v-else class="text-gray">未发布</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">出租状态:</text>
					<text v-if="items.isFrozen == 1" class="left text-green">已出租</text>
					<text v-else class="text-gray">未出租</text>
				</view>
				<view>
					<text class="weight">地理位置:</text>
					<text class="left">{{ items.province + items.city + items.detailPosition }}</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">房租:</text>
					<text class="left">{{ items.price }}元/月</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">交租时间:</text>
					<text class="left">{{ items.time }}个月交一次钱</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">最少租期:</text>
					<text class="left">{{ items.leastTime }}个月</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">押金:</text>
					<text class="left">{{ items.deposit }}元</text>
				</view>
				<view>
					<text class="weight">规则:</text>
					<text class="left">{{ items.rule }}</text>
				</view>
				<view>
					<text class="weight">介绍:</text>
					<text class="left">{{ items.introduction }}</text>
				</view>
				<view style="margin-top: 20rpx;">
					<view class="flex justify-between">
						<button class="cu-btn round bg-yellow" @click="modifyRent(items)">修改信息</button>
						<button class="cu-btn round bg-red" @click="deleteRent(items)">删除信息</button>
					</view>
				</view>
				<view style="margin-top: 20rpx;">
					<view class="flex justify-between">
						<button class="cu-btn round bg-yellow" @click="upRent(items)">发布出租</button>
						<button class="cu-btn round bg-red" @click="downRent(items)">下架出租</button>
					</view>
				</view>
			</uni-card>
		</view>
		<uni-pagination :show-icon="true" :total="total" :current="query.page" :pageSize="query.size" @change="change" class="pagination"></uni-pagination>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
import uniPagination from '@dcloudio/uni-ui/lib/uni-pagination/uni-pagination.vue';
export default {
	data() {
		return {
			query: {
				page: 1,
				size: 2
			},
			total: 0,
			rentList: [],
			access_token: '',
			userInfo: {}
		};
	},
	onLoad() {
		_this = this;
	},
	onShow() {
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
		let ok = this.checkToken(this.access_token);
		if (ok) {
			this.getRent();
		}
	},
	components: {
		uniCard,
		uniPagination
	},
	onBackPress(e) {
		if (e.from === 'backbutton') {
			uni.navigateBack({
				delta: 1
			});
			return true;
		}
	},
	onPullDownRefresh() {
		this.query.page = 1;
		this.getRent();
		uni.stopPullDownRefresh();
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
					title: '服务器开小差了'
				});
			}
		},
		change(e) {
			this.query.page = e.current;
			this.getRent();
		},
		getRent() {
			uni.request({
				url: http.BaseUrl + 'rent/findMyRent',
				data: _this.query,
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: function(res) {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.rentList = res.data.data.list;
						_this.total = res.data.data.total;
					}
				}
			});
		},
		modifyRent(items) {
			if (items.isFrozen == 1) {
				uni.showToast({
					title: '已经出租不能修改信息',
					icon: 'none'
				});
			} else {
				if (items.isOnRent == 0) {
					uni.navigateTo({
						url: '../rentInfo/modifyRent?hid=' + items.hid
					});
				} else {
					uni.showToast({
						title: '已经发布不能修改信息',
						icon: 'none'
					});
				}
			}
		},
		deleteRent(items) {
			if (items.isFrozen === 1) {
				uni.showToast({
					title: '已经出租不能修改信息',
					icon: 'none'
				});
			} else {
				uni.showModal({
					title: '确定要删除吗？',
					content: '删除不可逆哦',
					success: function(res) {
						if (res.confirm) {
							uni.showLoading({
								title: '请稍后'
							});
							uni.request({
								url: http.BaseUrl + 'rent/delRent/' + items.hid,
								method: 'DELETE',
								header: {
									Authorization: 'bearer ' + _this.access_token
								},
								complete: function(res) {
									uni.hideLoading();
									_this.checkResponse(res);
									if (res.data.code === 200) {
										uni.showToast({
											title: res.data.data,
											icon: 'success'
										});
										_this.getRent();
									}
								}
							});
						}
					}
				});
			}
		},
		upRent(item) {
			if (item.isFrozen == 1) {
				uni.showToast({
					title: '已经出租，不能修改',
					icon: 'none'
				});
			} else {
				if (item.isOnRent == 1) {
					uni.showToast({
						title: '已发布',
						icon: 'none'
					});
				} else {
					uni.request({
						url: http.BaseUrl + 'house/findBasicHouse/' + item.hid,
						method: 'GET',
						header: {
							Authorization: 'bearer ' + _this.access_token
						},
						complete: res => {
							_this.checkResponse(res);
							if (res.data.code == 200) {
								if (res.data.data.isCheck == 2) {
									uni.showToast({
										title: '房屋未通过审核，请修改信息',
										icon: 'none'
									});
								} else if (res.data.data.isCheck == 0) {
									uni.showToast({
										title: '房屋正在审核',
										icon: 'none'
									});
								} else {
									uni.request({
										url: http.BaseUrl + 'rent/upRent/' + item.hid,
										method: 'PUT',
										header: {
											Authorization: 'bearer ' + _this.access_token
										},
										complete: res => {
											_this.checkResponse(res);
											if (res.data.code == 200) {
												uni.showToast({
													title: '上架成功',
													icon: 'success'
												});
												_this.getRent();
											}
										}
									});
								}
							}
						}
					});
				}
			}
		},
		downRent(item) {
			if (item.isFrozen == 1) {
				uni.showToast({
					title: '已经出租，不能修改',
					icon: 'none'
				});
			} else {
				if (item.isOnRent == 0) {
					uni.showToast({
						title: '已下架',
						icon: 'none'
					});
				} else {
					uni.request({
						url: http.BaseUrl + 'rent/downRent/' + item.hid,
						method: 'PUT',
						header: {
							Authorization: 'bearer ' + _this.access_token
						},
						complete: res => {
							_this.checkResponse(res);
							if (res.data.code == 200) {
								uni.showToast({
									title: '下架成功',
									icon: 'success'
								});
								_this.getRent();
							}
						}
					});
				}
			}
		}
	}
};
</script>

<style scoped>
.pagination {
	position: fixed;
	bottom: 45rpx;
	width: 100%;
}
.weight {
	font-weight: bold;
}
.left {
	margin-left: 20rpx;
}
</style>
