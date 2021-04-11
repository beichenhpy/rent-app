<template>
	<view>
		<view class="cu-bar bg-gray">
			<view class="action" @tap="handleTap('picker1')">
				<text v-model="text">{{text}}</text>
				<text class="cuIcon-triangledownfill"></text>
			</view>
		</view>
		<lb-picker ref="picker1" v-model="selected" mode="multiSelector" :list="regionData" :level="level" :props="regionProps" @confirm="handleConfirm"></lb-picker>
	</view>
</template>

<script>
import http from '../../common/http.js';
export default {
	data() {
		return {
			regionData: [],
			selected: [],
			text:'选择所在区域',
			regionProps: {
				label: 'name',
				value: 'name',
				children: 'children'
			}
		};
	},
	props:{
		level:{
			type:Number,
			default:2
		}
	},
	created() {
	},
	methods: {
		getRegion() {
			var self = this;
			uni.request({
				url: http.BaseUrl + 'admin/findProvinces',
				success: function(res) {
					self.regionData = res.data.data;
				}
			});
		},
		handleTap(picker) {
			this.getRegion();
			this.$refs[picker].show();
		},
		// 拿到选择的值 list类型
		handleConfirm(item) {
			this.$emit('regions', item);
			this.text = item.value[1]
		}
	}
};
</script>

<style>
.chooseRegion {
	color: #aaa899;
	font-size: 30rpx;
	background-color: #ffffff;
	line-height: 97rpx;
}
.chooseRegion::after {
	border: none;
}
</style>
