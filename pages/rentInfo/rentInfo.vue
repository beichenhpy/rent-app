<template>
	<view style="margin-bottom: 20%;">
		<!-- 轮播 -->
		<swiper class="screen-swiper square-dot" :indicator-dots="true" :circular="true" :autoplay="true">
			<swiper-item v-for="(item, index) in imageList" :key="index" @click="previewPhoto"><image :src="item" mode="aspectFill"></image></swiper-item>
			<swiper-item><view @click="showVideo" style="text-align: center; font-size: 60rpx; margin-top: 150rpx; background-color: gray;">查看视频</view></swiper-item>
		</swiper>
		<!-- 房东信息 -->
		<view class="cu-list menu">
			<view class="cu-item"><view class="content">房东信息</view></view>
			<view class="cu-item">
				<view class="content">
					<image :src="owner.profilePhoto" mode="aspectFit" style="margin-right: 40rpx;"></image>
					<text class="text-grey">{{ owner.nickName }}</text>
				</view>
				<view class="action">
					<text class="text-grey">{{ owner.sex }}</text>
				</view>
			</view>
		</view>
		<!-- 基本信息 -->
		<uni-card title="基本信息" :is-shadow="true" mode="basic">
			<view style="margin-bottom: 20rpx;">
				<text class="cuIcon-home" style="margin-right: 20rpx;"></text>
				<text class="text-grey">{{ rentInfo.detailPosition }}</text>
			</view>
			<view style="margin-bottom: 20rpx;">
				<text class="cuIcon-moneybag" style="margin-right: 20rpx;"></text>
				<text class="text-grey">{{ rentInfo.price }}元/月</text>
			</view>
			<view>
				<text class="cuIcon-time" style="margin-right: 20rpx;"></text>
				<text class="text-grey">{{ rentInfo.time }}个月交一次钱</text>
			</view>
		</uni-card>
		<!-- 房屋详细信息 -->
		<uni-card title="房屋信息" :is-shadow="true" mode="basic">
			<view style="margin-bottom: 20rpx;">
				<text class="cuIcon-location" style="margin-right: 20rpx;"></text>
				<text class="text-grey">详细地址:{{ rentInfo.detailPosition }}{{ house.building }}栋{{ house.unit }}单元{{ house.houseNum }}房</text>
			</view>
			<view style="margin-bottom: 20rpx;">
				<text class="cuIcon-moreandroid" style="margin-right: 20rpx;"></text>
				<text class="text-grey">户型:{{ house.layout }}</text>
			</view>
			<view style="margin-bottom: 20rpx;">
				<text class="cuIcon-moreandroid" style="margin-right: 20rpx;"></text>
				<text class="text-grey">朝向:{{ house.forward }}</text>
			</view>
			<view>
				<text class="cuIcon-moreandroid" style="margin-right: 20rpx;"></text>
				<text class="text-grey">面积:{{ house.area }}</text>
			</view>
		</uni-card>
		<!-- 详细信息 -->
		<uni-card title="配套设施" :is-shadow="true" mode="basic">
			<view class="cu-list grid col-3 no-border">
				<view class="cu-item" v-for="(item,index) in src" :key="index">
					<image class="image-item" :src="item" mode="scaleToFill"></image>
				</view>
			</view>
		</uni-card>
		<!-- 介绍 -->
		<uni-card title="简介" :is-shadow="true" mode="basic">
			<view class="flex justify-center">
				<text class="text-black">{{rentInfo.introduction}}</text>
			</view>
		</uni-card>
		<!-- 描述 -->

		<!-- 底部 -->
		<view class="cu-bar bg-white tabbar border shop bottom_item">
			<button class="action" open-type="contact" @click="send">
				<view class="cuIcon-comment text-green"></view>
				留言
			</button>
			<view class="btn-group">
				<button class="cu-btn bg-orange round shadow-blur" @click="phone">电话联系</button>
				<button class="cu-btn bg-red round shadow-blur" @click="showAddModal">立即下单</button>
			</view>
		</view>
		<!-- 下单弹出框 -->
		<view>
			<view class="cu-modal" :class="addOrderModal ? 'show' : ''">
				<view class="cu-dialog">
					<view class="cu-bar bg-white justify-end"><view class="content">添加订单</view></view>
					<view class="padding-xl">
						<view class="cu-form-group margin-top">
							<view class="title">输入租期</view>
							<input placeholder="输入租期" type="number" v-model="months" />
						</view>
					</view>
					<view class="flex justify-center">
						<text @tap="isShowAgree" class="cuIcon" :class="showAgree ? 'cuIcon-radiobox' : 'cuIcon-round'" style="margin-right: 10rpx;">同意</text>
						<view @tap="ruleModal = true">房东规定</view>
					</view>
					<view class="cu-bar bg-white">
						<view class="action margin-0 flex-sub text-green solid-left" @tap="addOrderModal = false">取消</view>
						<view class="action margin-0 flex-sub  solid-left" @tap="addOrder">确定</view>
					</view>
				</view>
			</view>
		</view>
		<rule-modal :rule="rentInfo.rule" :ruleModal="ruleModal" @closeRuleModal="closeRuleModal"></rule-modal>
	</view>
</template>

<script>
var _this;
import ruleModal from '../../components/ruleModal/ruleModal.vue';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
import http from '../../common/http.js';
export default {
	data() {
		return {
			rentInfo: {},
			house:{},
			houseInfo:{},
			imageList:[],
			owner:{},
			months: 1,
			addOrderModal: false,
			showAgree: true,
			ruleModal: false,
			option:{},
			userInfo:{},
			access_token:'',
			src:[],
			robe:"../../static/house/robe.png",
			air:"../../static/house/air.png",
			bed:"../../static/house/bed.png",
			warm:"../../static/house/warm.png",
			wash:"../../static/house/wash.png",
			fridge:"../../static/house/fridge.png"
		};
	},
	onLoad(option) {
		_this = this;
		this.option = option;
		this.getRentInfo();
	},
	onShow() {
		this.userInfo =  uni.getStorageSync('userInfo');
		this.access_token = uni.getStorageSync('access_token');
	},
	onBackPress(e) {
		if (e.from === 'backbutton') {
			uni.navigateBack({
				delta: 1
			});
			return true;
		}
	},
	components: {
		uniCard,
		ruleModal
	},
	methods: {
		isShowAgree() {
			this.showAgree = !this.showAgree;
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
					title: '出租信息不存在'
				});
				uni.switchTab({
					url:"../index/index"
				})
			}
		},
		showImages(){
			if(_this.houseInfo.isWash === 1){
				_this.src.push(_this.wash)
			}
			if(_this.houseInfo.isRobe ===1){
				_this.src.push(_this.robe)
			}
			if(_this.houseInfo.isFridge ===1){
				_this.src.push(_this.fridge)
			}
			if(_this.houseInfo.isWarm ===1){
				_this.src.push(_this.warm)
			}
			if(_this.houseInfo.isAir ===1){
				_this.src.push(_this.air)
			}
			if(_this.houseInfo.isBed ===1){
				_this.src.push(_this.bed)
			}
		},
		getRentInfo() {
			uni.request({
				url: http.BaseUrl + 'rent/findRent',
				data: _this.option,
				method: 'GET',
				success: function(res) {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.rentInfo = res.data.data;
						_this.months = _this.rentInfo.leastTime;
						_this.house = res.data.data.house;
						_this.houseInfo = res.data.data.house.houseInfo;
						_this.showImages()
						_this.owner = res.data.data.owner;
						_this.imageList = res.data.data.house.houseInfo.images;
					}
				}
			});
		},
		previewPhoto() {
			uni.previewImage({
				current:0,
				urls: _this.imageList
			});
		},
		showVideo() {
			uni.navigateTo({
				url: 'video?video=' + _this.houseInfo.video
			});
		},
		phone() {
			let phone = this.owner.phone;
			uni.makePhoneCall({
				phoneNumber: phone
			});
		},
		showAddModal() {
			let ok = this.checkToken(_this.access_token);
			if (ok) {
				if (_this.userInfo.success === 0) {
					uni.showModal({
						title:"您还未进行实名认证",
						content:"是否前去进行实名认证，只有实名认证后才可以执行此操作",
						success: (res) => {
							if(res.confirm){
								uni.redirectTo({
									url: '../Authorization/Authorization?to=2&username=' + _this.userInfo.username
								});
							}
						}
					})
				}else{
					if (_this.userInfo.uid === this.owner.uid) {
						uni.showToast({
							title: '不能租自己的房子哟',
							icon: 'none'
						});
					} else {
						this.addOrderModal = true;
					}
				}
			}
		},
		addOrder() {
			//判断输入的months 是否 >= time
			if (this.months < this.rentInfo.leastTime) {
				uni.showToast({
					title: '租期不能小于最小租期',
					icon: 'none'
				});
			} else {
				if (!this.showAgree) {
					uni.showToast({
						title: '选择同意才能继续',
						icon: 'none'
					});
				} else {
					uni.showLoading({
						title: '正在下单'
					});
					uni.request({
						url: http.BaseUrl + 'order/addOrder',
						method: 'POST',
						data: {
							months: _this.months,
							hid: _this.rentInfo.hid
						},
						header: {
							Authorization: 'bearer ' + _this.access_token
						},
						complete: function(res) {
							uni.hideLoading();
							_this.checkResponse(res);
							if (res.data.code === 200) {
								_this.addOrderModal = false;
								uni.showToast({
									title: '下单成功',
									icon: 'success'
								});
								setTimeout(function() {
									uni.switchTab({
										url: '../Order/Order'
									});
								}, 1000);
							}
						}
					});
					this.addOrderModal = false;
				}
			}
		},
		closeRuleModal(close) {
			if (close) {
				this.ruleModal = false;
			}
		},
		send(){
			let ok = this.checkToken(this.access_token);
			if(ok){
				if(_this.userInfo.uid !== this.owner.uid){
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
									url:"../comment/useComment?sender="+JSON.stringify(_this.owner)
								})
							}
						}
					})
				}else{
					uni.showToast({
						title:"不能和自己对话",
						icon:"none"
					})
				}
			}
		}
	}
};
</script>

<style scoped>
.bottom_item {
	width: 100%;
	position: fixed;
	z-index: 999;
	bottom: var(--window-bottom);
}
.image-item {
	height: 100rpx;
	width: 100rpx;
}
</style>
