<template>
	<view>
		<view class="cu-modal" :class="priceModal ? 'show' : ''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">查看金额详情</view>
				</view>
				<view class="padding-xl">
					<view class="flex justify-between">
						<text class="text-black weight">基本金额:</text>
						<text class="text-orange">{{priceInfo.basic}}元</text>
					</view>
					<view class="flex justify-between">
						<text class="text-black weight">水费:</text>
						<text class="text-orange">{{priceInfo.water === null ? 0 : priceInfo.water}}元</text>
					</view>
					<view class="flex justify-between">
						<text class="text-black weight">电费:</text>
						<text class="text-orange">{{priceInfo.elect === null ? 0 : priceInfo.elect}}元</text>
					</view>
				</view>
				<view class="cu-bar bg-white">
					<view class="action margin-0 flex-sub text-green solid-left" @tap="close">关 闭</view>
				</view>
			</view>
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
			priceModal:{
				type:Boolean,
				default:false
			},
			priceInfo:{
				type:Object
			}
		},
		methods:{
			close(){
				this.$emit("closeModal",true);
			}
		}
	}
</script>

<style scoped>
.weight{
	font-weight: bold;
}
</style>
