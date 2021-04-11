<template>
	<view>
		<uni-card title="订单信息" :is-shadow="true" note="true" :extra="order.isFinish === 1 ? '已完成' : '未完成'">
			<order-info :order="order" :type="type"></order-info>
			<template v-slot:footer>
				<view class="flex justify-center">
					<button class="cu-btn round bg-blue" style="margin-left: 20rpx;" @click="showBills">查看账单</button>
					<button class="cu-btn round bg-yellow" style="margin-left: 20rpx;" v-if="order.isFirstPay === 0" @click="cancelOrder">取消订单</button>
					<button class="cu-btn round bg-red" style="margin-left: 20rpx;" @click="deleteOrder">删除订单</button>
				</view>
			</template>
		</uni-card>
		<uni-card title="对应房屋信息" :is-shadow="true" note="true">
			<view class="flex margin">
				<text class="text-black" style="font-weight: bold;">地点:</text>
				<text class="text-gray" style="margin-left: 20rpx;">{{ house.province + house.city + house.village }}</text>
			</view>
			<view class="flex margin">
				<text class="text-black" style="font-weight: bold;">地址:</text>
				<text class="text-gray" style="margin-left: 20rpx;">{{ house.address + house.building + '栋' + house.unit + '单元' }}</text>
			</view>
			<view class="flex margin">
				<text class="text-black" style="font-weight: bold;">房间:</text>
				<text class="text-gray" style="margin-left: 20rpx;">{{ house.houseNum + '号' }}</text>
			</view>
			<template v-slot:footer>
				<view class="flex justify-center">
					<button v-if="order.isFinish == 0" class="cu-btn round bg-red" style="margin-left: 20rpx;" @click="finishOrder">退租</button>
				</view>
			</template>
		</uni-card>
		<uni-card title="房东信息" :is-shadow="true" note="true">
			<view class="flex justify-between">
				<text>电话</text>
				<text>{{owner.phone}}</text>
			</view>
			<view class="flex justify-between">
				<text>姓名</text>
				<text>{{idCardInfo.realName}}</text>
			</view>
			<template v-slot:footer>
				<view class="flex justify-center">
					<button class="cu-btn round bg-blue" style="margin-left: 20rpx;" @click="contact">电话联系</button>
					<button class="cu-btn round bg-red" style="margin-left: 20rpx;" @click="comment">在线交流</button>
				</view>
			</template>
		</uni-card>
		<view style="margin-top: 40%;">
			<view class="flex justify-start"><text class="text-red">注意:</text></view>
			<view class="flex justify-start"><text class="text-gray">如果想要确认支付成功请点击查看账单核对信息后确认，确认只有一次机会，请仔细核对</text></view>
		</view>
	</view>
</template>

<script>
var _this;
import orderInfo from '../../components/order/orderInfo.vue';
import http from '../../common/http.js';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
export default {
	data() {
		return {
			order: {},
			access_token: '',
			userInfo: {},
			house: {},
			type: 'renter',
			isBreak:0,
			finishContent:'您确认要退租吗？如果要退租，押金可以再押金管理中申请退还',
			owner:{},
			idCardInfo:{}
		};
	},
	onLoad(value) {
		_this = this;
		this.order.oid = value.oid;
		this.order.hid = value.hid;
	},
	onShow() {
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
		this.getOrderInfo();
	},
	onPullDownRefresh() {
		this.getOrderInfo();
		uni.stopPullDownRefresh();
	},
	components: {
		uniCard,
		orderInfo
	},
	onBackPress(e) {
		if (e.from === 'backbutton') {
			uni.navigateBack({
				delta: 1
			});
			return true;
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
					uni.navigateTo({
						url: '../login/reLogin?username=' + _this.userInfo.username
					});
				}, 1000);
			}
			if (res.data.code === 400 || res.data.status === 500) {
				uni.showToast({
					icon: 'none',
					title: '订单信息不存在'
				});
				uni.switchTab({
					url:"Order"
				})
			}
		},
		getOrderInfo() {
			uni.request({
				url: http.BaseUrl + 'order/findOrder',
				method: 'GET',
				data: {
					type: 'renter',
					oid: _this.order.oid
				},
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: function(res) {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.order = res.data.data;
						_this.house = res.data.data.house;
						_this.owner = res.data.data.user;
						_this.idCardInfo = res.data.data.user.idCardInfo;
					}
				}
			});
		},
		deleteOrder() {
			if (this.order.isFinish === 0) {
				uni.showToast({
					title: '订单未完成，不能删除',
					icon: 'none'
				});
			} else {
				uni.showModal({
					title: '正在删除订单',
					content: '您确定要删除订单吗?',
					success: res => {
						if (res.confirm) {
							uni.showLoading({
								title:"正在删除"
							})
							uni.request({
								url: http.BaseUrl + 'order/delOrder/' + _this.order.oid + '?type=renter',
								method: 'DELETE',
								header: {
									Authorization: 'bearer ' + _this.access_token
								},
								complete: res => {
									uni.hideLoading()
									_this.checkResponse(res);
									if (res.data.code === 200) {
										uni.showToast({
											title: '删除成功',
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
				});
			}
		},
		showBills() {
			uni.navigateTo({
				url: 'bills?oid=' + _this.order.oid + '&type=renter'
			});
		},
		cancelOrder() {
			uni.showModal({
				title: '正在取消订单',
				content: '您确定要取消订单吗?',
				success: res => {
					if (res.confirm) {
						uni.showLoading({
							title:"正在删除"
						})
						uni.request({
							url: http.BaseUrl + 'order/cancel/' + _this.order.oid,
							method: 'DELETE',
							header: {
								Authorization: 'bearer ' + _this.access_token
							},
							complete: res => {
								uni.hideLoading()
								_this.checkResponse(res);
								if (res.data.code === 200) {
									uni.showToast({
										title: '取消成功',
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
			});
		},
		getTime: function(endTime) {
			let date = new Date(),
				year = date.getFullYear(),
				month = date.getMonth() + 1,
				day = date.getDate(),
				hour = date.getHours() < 10 ? '0' + date.getHours() : date.getHours(),
				minute = date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes(),
				second = date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds();
			month >= 1 && month <= 9 ? (month = '0' + month) : '';
			day >= 0 && day <= 9 ? (day = '0' + day) : '';
			endTime = endTime.split("-");
			let months = parseInt(endTime[0])*12+parseInt(endTime[1]);
			let now = parseInt(year*12)+parseInt(month);
			if(months - now > 0){
				this.isBreak = 1;
			}
		},
		finishOrder() {
			this.getTime(this.order.endTime);
			if (_this.order.isFinish === 0) {
				uni.request({
					url: http.BaseUrl + 'order/findNoPay/' + _this.order.oid,
					method: 'GET',
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					success: res => {
						_this.checkResponse(res);
						if (res.data.code === 200) {
							if (res.data.data === 1) {
								uni.showToast({
									title: '不能退租，您还有未完成的账单',
									icon: 'none'
								});
							} else {
								if (_this.order.renterConfirm == 0 || _this.order.ownerConfirm == 0) {
									uni.showToast({
										title: '合同未完成不能退租',
										icon: 'none'
									});
								} else {
									if(_this.isBreak == 1){
										_this.finishContent = '您提前退租会导致无法进行退还押金并且需要按照合同赔偿，是否继续？'
									}
									uni.showModal({
										title: '正在退租操作',
										content: _this.finishContent,
										success: res => {
											if (res.confirm) {
												uni.request({
													url: http.BaseUrl + 'order/finishOrder',
													data: _this.order,
													method: 'PUT',
													header: {
														Authorization: 'bearer ' + _this.access_token
													},
													success: res => {
														_this.checkResponse(res);
														if (res.data.code === 200) {
															uni.showToast({
																title: '退租成功',
																icon: 'success'
															});
															uni.navigateBack({
																delta: 1
															});
														}
														if(res.data.code === 440){
															uni.showToast({
																title: '退租成功,押金无法退回',
																icon: 'none'
															});
															uni.navigateBack({
																delta: 1
															});
														}
													}
												});
											}
										}
									});
								}
							}
						}
					}
				});
			} else {
				uni.showToast({
					title: '您的订单已完成，无需退租',
					icon: 'none'
				});
			}
		},
		contact(){
			let phone = this.owner.phone;
			uni.makePhoneCall({
				phoneNumber: phone
			});
		},
		comment(){
			let owner1 = {
				uid:_this.owner.uid,
				nickName:_this.owner.nickName,
				profilePhoto:_this.owner.profilePhoto,
				unRead:0
			}
			uni.request({
				url:http.BaseUrl + 'comment/addFriend',
				data:{
					uid:_this.userInfo.uid,
					friendUid:_this.owner.uid
				},
				method:"POST",
				success: (res) => {
					_this.checkResponse(res);
					if(res.data.code === 200){
						uni.navigateTo({
							url:"../comment/useComment?sender="+JSON.stringify(owner1)
						})
					}
				}
			})
		}
	}
};
</script>

<style scoped></style>
