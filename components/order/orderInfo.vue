<template>
	<view>
		<view class="flex justify-between margin">
			<text class="text-black">开始时间:</text>
			<text class="text-grey">{{ order.startTime }}</text>
		</view>
		<view class="flex justify-between margin">
			<text class="text-black">结束时间:</text>
			<text class="text-grey">{{ order.endTime }}</text>
		</view>
		<view class="flex justify-between margin">
			<text class="text-black">租期:</text>
			<text class="text-grey">{{ order.months }}个月</text>
		</view>
		<view class="flex justify-start margin">
			<text class="text-black">租金:</text>
			<text class="text-grey">{{ order.price }}元/月</text>
		</view>
		<view class="flex justify-between margin">
			<text class="text-black">押金:<text class="text-grey">{{ order.deposit }}元</text>
			</text>
			<button class="cu-btn bg-blue" v-if="type === 'renter'" @click="showDeposit(order.oid,order.isFinish)">查看押金信息</button>
		</view>
		<view class="flex justify-between margin">
			<text class="text-black">合同:</text>
			<button class="cu-btn bg-blue" @click="showContract(order,type)">合同管理</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			};
		},
		props:{
			order:{
				type:Object
			},
			type:{
				type:String
			}
		},
		methods:{
			showDeposit(oid,isFinish){
				uni.navigateTo({
					url:"../../pages/Order/deposit?oid="+oid+"&isFinish="+isFinish
				})
			},
			showContract(order,type){
				uni.navigateTo({
					url:"../../pages/Order/contract?oid="+order.oid+"&type="+type+"&isFinish="+order.isFinish
				})
			}
		}
	}
</script>

<style scoped>
.margin {
	margin-top: 30rpx;
	font-size: 30rpx;
}
</style>
