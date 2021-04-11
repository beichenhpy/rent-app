<template>
	<view style="margin-bottom: 20%;">
		<house-form :house="house" 
		@getRegion="getRegion" 
		@getForward="getForward" 
		@getLayout="getlayout" 
		@getAddress="getAddress"
		@getHouseNum="getHouseNum"
		@getBuilding="getBuilding"
		@getUnit="getUnit"
		@getArea="getArea"></house-form>
		<!-- 底部 -->
		<view class="cu-bar bg-white tabbar border shop bottom_item">
			<view class="btn-group">
				<button class="cu-btn bg-grey round shadow-blur" @click="cancle">取消</button>
				<button class="cu-btn bg-red round shadow-blur" @click="addHouse">继续添加</button>
			</view>
		</view>
	</view>
</template>

<script>
var _this;
import houseForm from '../../components/house/houseForm.vue'
import regionPicker from '../../components/RegionPicker/RegionPicker.vue';
import http from '../../common/http.js';
export default {
	data() {
		return {
			house: {
				province: '广西壮族自治区',
				city: '南宁市',
				village: '西乡塘区',
				address: '',
				building: '',
				unit: '',
				houseNum: '',
				layout: '1室0厅0卫',
				forward: '东',
				area: ''
			},
			forwards: ['东', '南', '西', '北', '东南', '西南', '东北', '西北', '南北', '东西'],
			forwardSelected: '',
			layouts: [
				['1室', '2室', '3室', '4室', '5室', '6室', '7室', '8室', '9室'],
				['0厅', '1厅', '2厅', '3厅', '4厅', '5厅', '6厅', '7厅', '8厅', '9厅'],
				['0卫', '1卫', '2卫', '3卫', '4卫', '5卫', '6卫', '7卫', '8卫', '9卫']
			],
			layoutsSelected: [],
			unInput: true,
			access_token:'',
			userInfo:{}
		};
	},
	onLoad() {
		_this = this;
	},
	onShow() {
		this.userInfo =  uni.getStorageSync('userInfo');
		this.access_token = uni.getStorageSync('access_token');
		if (_this.access_token === '') {
			uni.showToast({
				title: '未登录',
				icon: 'none'
			});
			setTimeout(function() {
				uni.navigateTo({
					url: '../login/login'
				});
			}, 1000);
		} else {
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
						if(res.cancel){
							uni.navigateBack({
								delta:1
							})
						}
					}
				});
			}
		}
	},
	components: {
		regionPicker,
		houseForm
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
			if(res.data.code === 450){
				uni.showToast({
					title:"该房屋已经被添加",
					icon:"none"
				})
			}
		},
		getRegion(e) {
			this.house.province = e.value[0]
			this.house.city = e.value[1]
			this.house.village = e.value[2]
		},
		getForward(e) {
			this.house.forward = e.value;
		},
		showForward() {
			this.$refs['picker1'].show();
		},
		showlayout() {
			this.$refs['picker2'].show();
		},
		getlayout(e) {
			this.house.layout = e;
		},
		checkAddress(){
			return /^\d+$/.test(this.house.address) || this.house.address.length <4 ? false :true;
		},
		checkBuilding(){
			return !/^\d+$/.test(this.house.building) ? false :true;
		},
		checkUnit(){
			return !/^\d+$/.test(this.house.unit)? false :true;
		},
		checkHouseNum(){
			return !/^\d+$/.test(this.house.houseNum) ? false :true;
		},
		checkArea(){
			return !/^[0-9]+(.[0-9])?$/.test(this.house.area)  || this.house.area.length < 2 ? false :true;
		},
		checkInput() {
			let addressOk = this.checkAddress();
			let buildingOk = this.checkBuilding();
			let unitOk = this.checkUnit();
			let houseNumOk =  this.checkHouseNum();
			let areaOk = this.checkArea();
			if(!(addressOk && buildingOk && unitOk && houseNumOk && areaOk)){
				uni.showToast({
					title: '请修改红色部分内容',
					icon: 'none'
				});
				this.unInput = true;
			}else{
				this.unInput = false;
			}
		},
		getHouseNum(e){
			this.house.houseNum = e;
		},
		getBuilding(e){
			this.house.building = e;
		},
		getUnit(e){
			this.house.unit = e;
		},
		getArea(e){
			this.house.area = e;
		},
		getAddress(e){
			this.house.address = e;
		},
		addHouse() {
			uni.showLoading({
				title: '正在添加，请稍后'
			});
			_this.checkInput();
			if (!this.unInput) {
				uni.request({
					url: http.BaseUrl + 'house/modifyHouse',
					method: 'POST',
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					data: _this.house,
					complete: function(res) {
						uni.hideLoading();
						_this.checkResponse(res);
						if (res.data.code === 200) {
							uni.showModal({
								title: '添加成功',
								content: '是否继续添加详细信息',
								success: function(res) {
									if (res.confirm) {
										uni.redirectTo({
											url:"house"
										})
									}else{
										uni.switchTab({
											url:"../index/index"
										})
									}
								}
							});
						}
					}
				});
			}else{
				uni.showToast({
					title: '请修改红色部分内容',
					icon: 'none'
				});
			}
		},
		cancle(){
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
		}
	}
};
</script>

<style scoped>
.weight {
	font-weight: bold;
	font-size: 33rpx;
}
.bottom_item {
	width: 100%;
	position: fixed;
	z-index: 999;
	bottom: var(--window-bottom);
}
.warn {
	border-bottom: 2rpx solid red;
}
</style>
