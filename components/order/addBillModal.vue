<template>
	<view>
		<view class="cu-modal" :class="addBillModal ? 'show' : ''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">添加房租</view>
				</view>
				<view>
					<view class="flex justify-between cu-form-group">
						<view>房屋电表读数</view>
						<input type="number" v-model="price.elect" @blur="checkInput(price.elect)"/>度
					</view>
					<view class="flex justify-between cu-form-group">
						<view>电车电表读数</view>
						<input  type="number" v-model="price.electCar" @blur="checkInput(price.electCar)"/>度
					</view>
					<view class="flex justify-between cu-form-group">
						<view>房屋水表读数</view>
						<input  type="number" v-model="price.water" @blur="checkInput(price.water)"/>吨
					</view>
				</view>
				<view class="cu-bar bg-white">
					<view class="action margin-0 flex-sub text-green solid-left" @tap="close">关 闭</view>
					<view class="action margin-0 flex-sub text-green solid-left" @tap="add">添 加</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				price:{
					elect:'',
					electCar:'',
					water:''
				}
				
			};
		},
		props:{
			addBillModal:{
				type:Boolean,
				default:false
			},
			oid:{
				type:String
			}
		},
		methods:{
			close(){
				this.$emit("closeModal",true);
			},
			add(){
				this.$emit("addBill",this.price);
			},
			checkInput(item){
				if(item.indexOf(".")!= -1){
					uni.showToast({
						title:"不能输入小数",
						icon:"none"
					})
					
				}
			}
		}
	}
</script>

<style>

</style>
