<template>
	<view style="margin-bottom: 20%;">
		<view class="flex justify-end" style="color: red; margin-right: 10rpx;">带*代表需要选择</view>
		<view class="flex justify-center top"><region-picker :level="3" @regions="getRegion"></region-picker></view>
		<form class="flex justify-center top">
			<view class="cu-form-group margin-top">
				<view class="title weight">省份*</view>
				<input name="province" :disabled="true" v-model="house.province"/>
			</view>
			<view class="cu-form-group margin-top">
				<view class="title weight">城市*</view>
				<input name="city" :disabled="true" v-model="house.city" />
			</view>
			<view class="cu-form-group margin-top">
				<view class="title weight">区县*</view>
				<input name="village" :disabled="true" v-model="house.village" />
			</view>
			<view class="cu-form-group margin-top" :class="addresswarn ? 'warn' : ''">
				<view class="title weight">小区</view>
				<input name="address" v-model="house.address" @blur="checkAddress(house.address)"/>
			</view>
			<view class="cu-form-group margin-top" :class="buildingwarn ? 'warn' : ''">
				<view class="title weight">楼栋</view>
				<input name="building" v-model="house.building" type="number" @blur="checkBuilding(house.building)"/>
				栋
			</view>
			<view class="cu-form-group margin-top" :class="unitwarn ? 'warn' : ''">
				<view class="title weight">单元</view>
				<input name="unit" v-model="house.unit" type="number" @blur="checkUnit(house.unit)"/>单元
			</view>
			<view class="cu-form-group margin-top" :class="houseNumwarn ? 'warn' : ''">
				<view class="title weight">房间号</view>
				<input name="houseNum" v-model="house.houseNum" type="number" @blur="checkHouseNum(house.houseNum)"/>
			</view>
			<view class="cu-form-group margin-top">
				<view class="title weight">户型*</view>
				<view @tap="showlayout"><input name="layout" v-model="house.layout" :disabled="true" /></view>
			</view>
			<view class="cu-form-group margin-top">
				<view class="title weight">朝向*</view>
				<view @tap="showForward"><input name="forward" v-model="house.forward" :disabled="true" /></view>
			</view>
			<view class="cu-form-group margin-top" :class="areawarn ? 'warn' : ''">
				<view class="title weight">面积</view>
				<input name="area" v-model="house.area" type="number" @blur="checkArea(house.area)"/>
				平(精确到后1位)
			</view>
		</form>
		<lb-picker ref="picker1" v-model="forwardSelected" mode="selector" :list="forwards" @confirm="getForward"></lb-picker>
		<lb-picker ref="picker2" v-model="layoutsSelected" mode="unlinkedSelector" :list="layouts" @confirm="getlayout"></lb-picker>
	</view>
</template>

<script>
import regionPicker from '../../components/RegionPicker/RegionPicker.vue';
import http from '../../common/http.js';
export default {
	data() {
		return {
			forwards: ['东', '南', '西', '北', '东南', '西南', '东北', '西北', '南北', '东西'],
			forwardSelected: '',
			layouts: [
				['1室', '2室', '3室', '4室', '5室', '6室', '7室', '8室', '9室'],
				['0厅', '1厅', '2厅', '3厅', '4厅', '5厅', '6厅', '7厅', '8厅', '9厅'],
				['0卫', '1卫', '2卫', '3卫', '4卫', '5卫', '6卫', '7卫', '8卫', '9卫']
			],
			layoutsSelected: [],
			addresswarn: false,
			buildingwarn: false,
			unitwarn: false,
			houseNumwarn: false,
			areawarn: false
		};
	},
	props:{
		house:{
			type:Object
		}
	},
	components: {
		regionPicker
	},
	methods: {
		getRegion(region) {
			this.$emit("getRegion",region)
		},
		getForward(forward) {
			this.$emit("getForward",forward)
		},
		showForward() {
			this.$refs['picker1'].show();
		},
		showlayout() {
			this.$refs['picker2'].show();
		},
		getlayout(layouts) {
			this.$emit("getLayout",layouts.item[0] + layouts.item[1] + layouts.item[2])
		},
		checkAddress(e){
			if (/^\d+$/.test(e) || e.length < 4) {
				this.addresswarn = true;
			} else {
				this.addresswarn = false;
			}
			this.$emit("getAddress",e)
		},
		checkBuilding(e){
			if (!/^\d+$/.test(e)) {
				this.buildingwarn = true;
			} else {
				this.buildingwarn = false;
			}
			this.$emit("getBuilding",e);
		},
		checkUnit(e){
			if (!/^\d+$/.test(e)) {
				this.unitwarn = true;
			} else {
				this.unitwarn = false;
			}
			this.$emit("getUnit",e);
		},
		checkHouseNum(e){
			if (!/^\d+$/.test(e) || e.length < 2) {
				this.houseNumwarn = true;
			} else {
				this.houseNumwarn = false;
			}
			this.$emit("getHouseNum",e);
		},
		checkArea(e){
			if (!/^[0-9]+(.[0-9])?$/.test(e)|| e.length < 2) {
				this.areawarn = true;
			} else {
				this.areawarn = false;
			}
			this.$emit("getArea",e);
		}
	}
};
</script>

<style scoped>
.weight {
	font-weight: bold;
	font-size: 33rpx;
}

.warn {
	border-bottom: 2rpx solid red;
}
</style>
