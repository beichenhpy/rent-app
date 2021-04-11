<template>
	<view>
		<view class="cu-modal" :class="ruleModal ? 'show' : ''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">查看规定</view>
				</view>
				<view class="padding-xl">
					<scroll-view>{{rule}}</scroll-view>
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
			ruleModal:{
				type:Boolean,
				default:false
			},
			rule:{
				type:String
			}
		},
		methods:{
			close(){
				this.$emit("closeRuleModal",true);
			}
		}
	}
</script>

<style>

</style>
