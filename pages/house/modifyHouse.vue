<template>
	<view style="margin-bottom: 20%;">
		<house-form
			:house="house"
			@getRegion="getRegion"
			@getForward="getForward"
			@getLayout="getlayout"
			@getAddress="getAddress"
			@getHouseNum="getHouseNum"
			@getBuilding="getBuilding"
			@getUnit="getUnit"
			@getArea="getArea"
		></house-form>
		<!-- 底部 -->
		<view class="cu-bar bg-white tabbar border shop bottom_item">
			<view class="btn-group">
				<button class="cu-btn bg-grey round shadow-blur" @click="cancle">取消</button>
				<button class="cu-btn bg-red round shadow-blur" @click="modifyHouse">修改</button>
			</view>
		</view>
	</view>
</template>

<script>
var _this;
import houseForm from '../../components/house/houseForm.vue';
import http from '../../common/http.js';
export default {
	data() {
		return {
			house: {},
			unInput: true,
			access_token: '',
			userInfo: {}
		};
	},
	components: {
		houseForm
	},
	onLoad(value) {
		this.house = JSON.parse(value.house);
		console.log(this.house)
		_this = this;
	},
	onShow() {
		this.userInfo = uni.getStorageSync('userInfo');
		this.access_token = uni.getStorageSync('access_token');
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
		checkAddress() {
			if(/^\d+$/.test(this.house.address) || this.house.address.length < 4){
				return false;
			}else{
				return true;
			}
		},
		checkBuilding() {
			if(!/^\d+$/.test(this.house.building)){
				return false;
			}else{
				return true;
			}
		},
		checkUnit() {
			if(!/^\d+$/.test(this.house.unit)){
				return false
			}else{
				return true;
			}
		},
		checkHouseNum() {
			console.log(this.house.houseNum)
			if(!/^\d+$/.test(this.house.houseNum) || this.house.houseNum.length < 2){
				return false;
			}else{
				return true;
			}
		},
		checkArea() {
			if(!/^[0-9]+(.[0-9])?$/.test(this.house.area) || this.house.area.length < 2){
				return false;
			}else{
				return true;
			}
		},
		checkInput() {
			let addressOk = this.checkAddress();
			let buildingOk = this.checkBuilding();
			let unitOk = this.checkUnit();
			let houseNumOk = this.checkHouseNum();
			let areaOk = this.checkArea();
			if (!(addressOk && buildingOk && unitOk && houseNumOk && areaOk)) {
				this.unInput = true;
				uni.showToast({
					title: '请修改红色部分内容',
					icon: 'none'
				});
			} else {
				this.unInput = false;
			}
		},
		getHouseNum(e) {
			this.house.houseNum = e;
		},
		getBuilding(e) {
			this.house.building = e;
		},
		getUnit(e) {
			this.house.unit = e;
		},
		getArea(e) {
			this.house.area = e;
		},
		getAddress(e) {
			this.house.address = e;
		},
		getRegion(e) {
			this.house.province = e.value[0];
			this.house.city = e.value[1];
			this.house.village = e.value[2];
		},
		getForward(e) {
			this.house.forward = e.value;
		},
		getlayout(e) {
			this.house.layout = e;
		},
		modifyHouse() {
			this.checkInput();
			console.log(this.unInput)
			if (!this.unInput) {
				uni.showLoading({
					title: '正在修改，请稍后'
				});
				uni.request({
					url: http.BaseUrl + 'house/updateHouse',
					method: 'PUT',
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					data: _this.house,
					complete: function(res) {
						uni.hideLoading();
						_this.checkResponse(res);
						if (res.data.code === 200) {
							uni.showToast({
								title: '修改成功',
								icon: 'success'
							});
							setTimeout(function() {
								uni.navigateBack({
									delta: 1
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
		},
		cancle() {
			uni.navigateBack({
				delta: 1
			});
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
</style>
