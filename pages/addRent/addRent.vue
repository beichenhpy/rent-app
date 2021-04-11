<template>
	<view style="margin-bottom: 20%;">
		<form ref="rentForm">
			<view class="cu-form-group margin-top" :class="titleWarn ? 'warn' : ''">
				<view class="title weight">标题</view>
				<input name="title" v-model="rent.title" @blur="checkTitle" placeholder="限制20字以内" />
			</view>
			<view class="cu-form-group margin-top" :class="priceWarn ? 'warn' : ''">
				<view class="title weight">租金</view>
				<input name="price" v-model="rent.price" @blur="checkPrice" type="number" placeholder="请输入整数" />
				元/月
			</view>
			<view class="cu-form-group margin-top" :class="depositWarn ? 'warn' : ''">
				<view class="title weight">基本押金</view>
				<input name="deposit" v-model="rent.deposit" @blur="checkDeposit" type="number" placeholder="请输入整数" />
				元
			</view>
			<view class="cu-form-group margin-top" :class="keyDepositWarn ? 'warn' : ''">
				<view class="title weight">钥匙押金</view>
				<input name="keyDeposit" v-model="rent.keyDeposit" @blur="checkKeyDeposit" type="number" placeholder="请输入整数" />
				元
			</view>
			<view class="cu-form-group margin-top" :class="timeWarn ? 'warn' : ''">
				<view class="title weight">交租日期</view>
				<view @tap="showTimePicker"><input name="time" v-model="rent.time" :disabled="true" type="number" placeholder="点击选择" /></view>
				个月
			</view>
			<view class="cu-form-group margin-top" :class="leastTimeWarn ? 'warn' : ''">
				<view class="title weight">最少租期</view>
				<view @tap="showLeastTimePicker"><input name="leastTime" v-model="rent.leastTime" :disabled="true" type="number" placeholder="点击选择" /></view>
				个月
			</view>
			<view class="cu-form-group margin-top" :class="houseWarn ? 'warn' : ''">
				<view class="title weight">选择房屋</view>
				<input v-model="house" :disabled="true" placeholder="点击选择" @click="chooseHouse" />
			</view>
			<!-- !!!!! placeholder 在ios表现有偏移 建议使用 第一种样式 -->
			<view class="cu-form-group margin-top" :class="introductionWarn ? 'warn' : ''">
				<text class="title weight">输入介绍</text>
				<textarea name="introduction" maxlength="100" v-model="rent.introduction" @blur="checkIn" placeholder="限制100字以内"></textarea>
			</view>

			<!-- !!!!! placeholder 在ios表现有偏移 建议使用 第一种样式 -->
			<view class="cu-form-group margin-top" :class="RuleWarn ? 'warn' : ''">
				<text class="title weight">输入规则</text>
				<textarea name="rule" maxlength="100" v-model="rent.rule" @blur="checkRule" placeholder="限制100字以内"></textarea>
			</view>
		</form>
		<!-- 底部 -->
		<view class="cu-bar bg-white tabbar border shop bottom_item">
			<view class="btn-group">
				<button class="cu-btn bg-grey round shadow-blur" @click="cancle">取消</button>
				<button class="cu-btn bg-red round shadow-blur" @click="addRent">添加</button>
			</view>
		</view>
		<lb-picker ref="picker1" mode="selector" :list="times" @confirm="getTime"></lb-picker>
		<lb-picker ref="picker2" mode="selector" :list="leasttimes" @confirm="getLeasttime"></lb-picker>
		<lb-picker ref="picker3" mode="selector" :list="houseChecked" @confirm="getHouseCheck"></lb-picker>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
export default {
	data() {
		return {
			rent: {
				title: '',
				price: '',
				deposit: '',
				keyDeposit: '',
				time: '',
				rule: '',
				hid: '',
				introduction: '',
				leastTime: ''
			},
			userInfo: {},
			access_token: '',
			confirmOk: false,
			titleWarn: false,
			introductionWarn: false,
			RuleWarn: false,
			priceWarn: false,
			depositWarn: false,
			keyDepositWarn: false,
			timeWarn: false,
			leastTimeWarn: false,
			houseWarn: false,
			times: [
				{
					label: '一个月',
					value: 1
				},
				{
					label: '两个月',
					value: 2
				},
				{
					label: '三个月',
					value: 3
				},
				{
					label: '六个月',
					value: 6
				}
			],
			leasttimes: [
				{
					label: '一个月',
					value: 1
				},
				{
					label: '二个月',
					value: 2
				},
				{
					label: '三个月',
					value: 3
				},
				{
					label: '四个月',
					value: 4
				},
				{
					label: '五个月',
					value: 5
				},
				{
					label: '六个月',
					value: 6
				},
				{
					label: '七个月',
					value: 7
				},
				{
					label: '八个月',
					value: 8
				},
				{
					label: '九个月',
					value: 9
				},
				{
					label: '十个月',
					value: 10
				},
				{
					label: '十一个月',
					value: 11
				},
				{
					label: '一年',
					value: 12
				}
			],
			houseChecked: [],
			house: ''
		};
	},
	onLoad() {
		_this = this;
	},
	onShow() {
		this.userInfo = uni.getStorageSync('userInfo');
		this.access_token = uni.getStorageSync('access_token');
		if (this.access_token === '') {
			uni.navigateTo({
				url: '../login/login'
			});
		} else {
			if (this.userInfo.success === 0) {
				uni.showModal({
					title: '您还未进行实名认证',
					content: '是否前去进行实名认证，只有实名认证后才可以执行此操作',
					success: res => {
						if (res.confirm) {
							uni.redirectTo({
								url: '../Authorization/Authorization?to=2&username=' + _this.userInfo.username
							});
						}
						if (res.cancel) {
							uni.navigateBack({
								delta: 1
							});
						}
					}
				});
			} else {
				this.getHouse();
			}
		}
	},
	onBackPress(e) {
		if (e.from === 'backbutton') {
			uni.showModal({
				title: '您确定要离开吗?',
				content: '未添加成功，添加记录会丢失',
				success: function(res) {
					if (res.confirm) {
						uni.switchTab({
							url: '../index/index'
						});
					}
				}
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
					title: '服务器开小差了'
				});
			}
		},
		checkTitle() {
			if (this.rent.title === '' || this.rent.title.length >= 20) {
				this.titleWarn = true;
				return false;
			} else {
				this.titleWarn = false;
				return true;
			}
		},
		checkIn() {
			if (this.rent.introduction === '' || this.rent.introduction >= 100) {
				this.introductionWarn = true;
				return false;
			} else {
				this.introductionWarn = false;
				return true;
			}
		},
		checkRule() {
			if (this.rent.rule === '' || this.rent.rule >= 100) {
				this.RuleWarn = true;
				return false;
			} else {
				this.RuleWarn = false;
				return true;
			}
		},
		checkPrice() {
			if (Number(this.rent.price) <= 0 || this.rent.price === '' || this.rent.price.indexOf('.') != -1) {
				this.priceWarn = true;
				return false;
			} else {
				this.priceWarn = false;
				return true;
			}
		},
		checkDeposit() {
			if (Number(this.rent.deposit) <= 0 || this.rent.deposit === '' || this.rent.deposit.indexOf('.') != -1) {
				this.depositWarn = true;
				return false;
			} else {
				this.depositWarn = false;
				return true;
			}
		},
		checkKeyDeposit() {
			if (Number(this.rent.keyDeposit) <= 0 || this.rent.keyDeposit === '' || this.rent.keyDeposit.indexOf('.') != -1) {
				this.keyDepositWarn = true;
				return false;
			} else {
				this.keyDepositWarn = false;
				return true;
			}
		},
		checkTime() {
			if (this.rent.time === '') {
				this.timeWarn = true;
				return false;
			} else {
				this.timeWarn = false;
				return true;
			}
		},
		checkLeastTime() {
			if (this.rent.leastTime === '' || Number(this.rent.leastTime) < Number(this.rent.time)) {
				uni.showToast({
					title:"交租日期不能小于最少租期",
					icon:"none"
				})
				this.leastTimeWarn = true;
				return false;
			} else {
				this.leastTimeWarn = false;
				return true;
			}
		},
		checkHouse() {
			if (this.rent.hid === '') {
				this.houseWarn = true;
				return false;
			} else {
				this.houseWarn = false;
				return true;
			}
		},
		checkConfirm() {
			let titleOk = this.checkTitle();
			let timeOk = this.checkTime();
			let ruleOk = this.checkRule();
			let introductionOk = this.checkIn();
			let leastTimeOk = this.checkLeastTime();
			let depositOk = this.checkDeposit();
			let keyDepositok = this.checkKeyDeposit();
			let houseok = this.checkHouse();
			let priceOk = this.checkPrice();
			if (titleOk && timeOk && ruleOk && introductionOk && leastTimeOk && depositOk && keyDepositok && houseok && priceOk) {
				this.confirmOk = true;
			}else{
				uni.showToast({
					title:"请修改红色部分内容",
					icon:"none"
				})
				this.confirmOk = false;
			}
		},
		showTimePicker() {
			this.$refs['picker1'].show();
		},
		getTime(e) {
			this.rent.time = e.value;
			this.rent.leastTime = '';
		},
		showLeastTimePicker() {
			let time = this.rent.time;
			if (time === '') {
				uni.showToast({
					title: '请先选择交租日期',
					icon: 'none'
				});
			} else {
				if (Number(time) == 2) {
					this.leasttimes = [
						{
							label: '二个月',
							value: 2
						},
						{
							label: '三个月',
							value: 3
						},
						{
							label: '四个月',
							value: 4
						},
						{
							label: '五个月',
							value: 5
						},
						{
							label: '六个月',
							value: 6
						},
						{
							label: '七个月',
							value: 7
						},
						{
							label: '八个月',
							value: 8
						},
						{
							label: '九个月',
							value: 9
						},
						{
							label: '十个月',
							value: 10
						},
						{
							label: '十一个月',
							value: 11
						},
						{
							label: '一年',
							value: 12
						}
					];
				}
				if(Number(time) == 3){
					this.leasttimes = [
						{
							label: '三个月',
							value: 3
						},
						{
							label: '四个月',
							value: 4
						},
						{
							label: '五个月',
							value: 5
						},
						{
							label: '六个月',
							value: 6
						},
						{
							label: '七个月',
							value: 7
						},
						{
							label: '八个月',
							value: 8
						},
						{
							label: '九个月',
							value: 9
						},
						{
							label: '十个月',
							value: 10
						},
						{
							label: '十一个月',
							value: 11
						},
						{
							label: '一年',
							value: 12
						}
					];
				}
				if(Number(time) == 6){
					this.leasttimes = [
						{
							label: '六个月',
							value: 6
						},
						{
							label: '七个月',
							value: 7
						},
						{
							label: '八个月',
							value: 8
						},
						{
							label: '九个月',
							value: 9
						},
						{
							label: '十个月',
							value: 10
						},
						{
							label: '十一个月',
							value: 11
						},
						{
							label: '一年',
							value: 12
						}
					];
				}
				this.$refs['picker2'].show();
			}
			
		},
		getLeasttime(e) {
			this.rent.leastTime = e.value;
			this.checkLeastTime();
		},
		cancle() {
			uni.showModal({
				title: '您确定要离开吗?',
				content: '未添加成功，添加记录会丢失',
				success: function(res) {
					if (res.confirm) {
						uni.switchTab({
							url: '../index/index'
						});
					}
				}
			});
		},
		chooseHouse() {
			this.$refs['picker3'].show();
		},
		getHouse() {
			uni.request({
				url: http.BaseUrl + 'house/findChecked',
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					_this.houseChecked = res.data.data;
				}
			});
		},
		getHouseCheck(e) {
			if (e !== null) {
				this.rent.hid = e.value;
				this.house = e.item.label;
			}
		},
		addRent() {
			this.checkConfirm();
			if (this.confirmOk) {
				uni.showLoading({
					title: '正在提交'
				});
				uni.request({
					url: http.BaseUrl + 'rent/putRent',
					method: 'POST',
					data: _this.rent,
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					complete: res => {
						uni.hideLoading();
						_this.checkResponse(res);
						if (res.data.code === 200) {
							uni.showToast({
								title: '添加成功',
								icon: 'success'
							});
							setTimeout(function() {
								uni.switchTab({
									url: '../index/index'
								});
							}, 500);
						}
					}
				});
			} else {
				uni.showToast({
					title: '请修改红色部分内容',
					icon: 'none'
				});
			}
		}
	}
};
</script>

<style scoped>
.weight {
	font-weight: bold;
}

.warn {
	border-bottom: 2rpx solid red;
}

.bottom_item {
	width: 100%;
	position: fixed;
	z-index: 999;
	bottom: var(--window-bottom);
}
</style>
