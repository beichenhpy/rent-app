<template>
	<view>
		<uni-card title="订单信息" :is-shadow="true" note="true" :extra="order.isFinish===1?'已完成':'未完成'">
			<order-info :order="order" :type="type"></order-info>
			<template v-slot:footer>
				<view class="flex justify-center">
					<button class="cu-btn round bg-blue" style="margin-left: 20rpx;" @click="showBills">查看账单</button>
					<button class="cu-btn round bg-red" style="margin-left: 20rpx;" @click="deleteOrder">删除订单</button>
				</view>
			</template>
		</uni-card>
		<uni-card title="对应房屋信息" :is-shadow="true">
			<view class="flex margin">
				<text class="text-black" style="font-weight: bold;">地点:</text>
				<text class="text-gray" style="margin-left: 20rpx;">{{house.province+house.city+house.village}}</text>
			</view>
			<view class="flex margin">
				<text class="text-black" style="font-weight: bold;">地址:</text>
				<text class="text-gray" style="margin-left: 20rpx;">{{house.address+house.building+'栋'+house.unit+'单元'}}</text>
			</view>
			<view class="flex margin">
				<text class="text-black" style="font-weight: bold;">房间:</text>
				<text class="text-gray" style="margin-left: 20rpx;">{{house.houseNum+'号'}}</text>
			</view>
		</uni-card>
		<uni-card title="租客信息" :is-shadow="true" note="true">
			<view class="flex justify-between">
				<text>电话</text>
				<text>{{renter.phone}}</text>
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
			<view class="flex justify-start">
				<text class="text-red">注意:</text>
			</view>
			<view class="flex justify-start">
				<text class="text-gray">如果想要确认支付成功请点击查看账单核对信息后确认，确认只有一次机会，请仔细核对</text>
			</view>
		</view>
	</view>
</template>

<script>
var _this;
import orderInfo from '../../components/order/orderInfo.vue'
import http from '../../common/http.js';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
export default {
	data() {
		return {
			order: {},
			house:{},
			renter:{},
			access_token: '',
			userInfo: {},
			type:'owner',
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
	components: {
		uniCard,
		orderInfo
	},
	onPullDownRefresh() {
		this.getOrderInfo()
		uni.stopPullDownRefresh()
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
					type: 'owner',
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
						_this.renter = res.data.data.user;
						_this.idCardInfo = res.data.data.user.idCardInfo;
					}
				}
			});
		},
		deleteOrder(){
			if(this.order.isFinish === 0){
				uni.showToast({
					title:"订单未完成，不能删除",
					icon:"none"
				})
			}else{
				uni.showModal({
					title:"正在删除订单",
					content:"您确定要删除订单吗?",
					success: (res) => {
						if(res.confirm){
							uni.showLoading({
								title:"正在删除"
							})
							uni.request({
								url:http.BaseUrl + 'order/delOrder/'+_this.order.oid+'?type=owner',
								method:"DELETE",
								header: {
									Authorization: 'bearer ' + _this.access_token
								},
								complete: (res) => {
									uni.hideLoading()
									_this.checkResponse(res);
									if(res.data.code === 200){
										uni.showToast({
											title:"删除成功",
											icon:"success"
										})
										uni.navigateBack({
											delta:1
										})
									}
								}
							})
						}
					}
				})
			}
		},
		showBills(){
			uni.navigateTo({
				url:"bills?oid="+_this.order.oid+"&type=owner"+"&isFinish="+_this.order.isFinish+"&hid="+_this.order.hid+"&billTime="+_this.order.billTime
			})
		},
		contact(){
			let phone = this.renter.phone;
			uni.makePhoneCall({
				phoneNumber: phone
			});
		},
		comment(){
			let renter1 = {
				uid:_this.renter.uid,
				nickName:_this.renter.nickName,
				profilePhoto:_this.renter.profilePhoto,
				unRead:0
			}
			uni.request({
				url:http.BaseUrl + 'comment/addFriend',
				data:{
					uid:_this.userInfo.uid,
					friendUid:_this.renter.uid
				},
				method:"POST",
				success: (res) => {
					_this.checkResponse(res);
					if(res.data.code === 200){
						uni.navigateTo({
							url:"../comment/useComment?sender="+JSON.stringify(renter1)
						})
					}
				}
			})
		}
	}
};
</script>

<style scoped>
	.margin {
		margin-top: 30rpx;
		font-size: 30rpx;
	}
</style>
