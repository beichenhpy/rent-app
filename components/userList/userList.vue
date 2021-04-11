<template>
	<view>
		<view class="cu-list menu">
			<view class="cu-item">
				<view class="content">
					<text class="cuIcon-btn text-olive"></text>
					<text class="text-grey">性别:{{ sex }}</text>
				</view>
				<view class="action">
					<button class="cu-btn round bg-red shadow" @click="changeSex">
						<text class="cuIcon-edit"></text>
						更改
					</button>
				</view>
			</view>
			<view class="cu-item">
				<button class="cu-btn content" @tap="forgetPassword">
					<text class="cuIcon-people text-olive"></text>
					<text class="text-grey">修改密码</text>
				</button>
			</view>
		</view>
		<view class="cu-list menu">
			<view class="cu-item">
				<button class="cu-btn content" @tap="house">
					<text class="cuIcon-home text-olive"></text>
					<text class="text-grey">房屋管理</text>
				</button>
			</view>
			<view class="cu-item">
				<button class="cu-btn content" @tap="rent">
					<text class="cuIcon-order text-olive"></text>
					<text class="text-grey">出租信息管理</text>
				</button>
			</view>
			<view class="cu-item">
				<button class="cu-btn content" @tap="renterdeposit">
					<text class="cuIcon-moneybag text-olive"></text>
					<text class="text-grey">租客押金管理</text>
				</button>
			</view>
			<view class="cu-item">
				<button class="cu-btn content" @tap="ownerdeposit">
					<text class="cuIcon-moneybag text-olive"></text>
					<text class="text-grey">房东押金管理</text>
				</button>
			</view>
			<view class="cu-item">
				<button class="cu-btn content" @tap="priceStatistics">
					<text class="cuIcon-moneybag text-olive"></text>
					<text class="text-grey">租金统计</text>
				</button>
			</view>
		</view>
		
		<lb-picker ref="picker" v-model="selected" mode="selector" :list="sexData" @confirm="handleConfirm"></lb-picker>
	</view>
</template>

<script>
	import http from '../../common/http.js'
export default {
	data() {
		return {
			newSex:'',
			sexData:['男','女'],
			selected:''
		};
	},
	props:{
		sex:{
			type:String
		}
	},
	methods: {
		checkToken(access_token) {
			if (access_token === '') {
				return false;
			} else {
				return true;
			}
		},
		checkResponse(res){
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
					let userInfo = uni.getStorageSync('userInfo');
					uni.navigateTo({
						url: '../login/reLogin?username=' + userInfo.username
					});
				}, 1000);
			}
			 if (res.data.code === 400) {
				uni.showToast({
					icon: 'none',
					title: res.data.data
				});
			}
		},
		forgetPassword(){
			let access_token = uni.getStorageSync('access_token');
			let ok  = this.checkToken(access_token);
			if(ok){
				uni.navigateTo({
					url:"../../pages/login/forget"
				})
			}else{
				uni.showToast({
					title:"尚未登录",
					icon:"none"
				})
			}
		},
		changeSex(sex){
			let access_token = uni.getStorageSync('access_token');
			let ok  = this.checkToken(access_token);
			if(ok){
				this.$refs.picker.show();
			}else{
				uni.showToast({
					title:"尚未登录",
					icon:"none"
				})
			}
		},
		handleConfirm(item){
			let access_token = uni.getStorageSync('access_token');
			let ok = this.checkToken(access_token)
			let that = this;
			if(ok){
				uni.request({
					url:http.BaseUrl + 'user/updateUser',
					header: {
						Authorization: 'bearer ' + access_token
					},
					method:"PUT",
					data:{
						type:"sex",
						profile:item.value
					},
					success:function(res){
						that.checkResponse(res);
						if(res.data.code === 200){
							that.$emit("changeSex",true);
						}
					}
				})
			}
		},
		house(){
			uni.navigateTo({
				url:"../../pages/house/house"
			})
		},
		rent(){
			uni.navigateTo({
				url:"../../pages/User/myRent"
			})
		},
		renterdeposit(){
			uni.navigateTo({
				url:"../../pages/User/MyDeposit"
			})
		},
		ownerdeposit(){
			uni.navigateTo({
				url:"../../pages/User/ownerDeposit"
			})
		},
		priceStatistics(){
			uni.navigateTo({
				url:"../../pages/User/priceStatistics"
			})
		}
	}
};
</script>

<style></style>
