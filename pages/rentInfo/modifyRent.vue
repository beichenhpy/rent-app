<template>
	<view>
		<uni-card title="基本信息" :is-shadow="true" mode="basic" note="true">
			<view class="weight" style="color: red;">仅显示可修改信息</view>
			<view>
				<form>
					<view class="cu-form-group fontSize">
						<text class="weight">标题:</text>
						<input :class="TitleOk ? 'noWarn' : 'warn'" v-model="rentInfo.title" @blur="checkTitle"/>
					</view>
					<view class="cu-form-group fontSize">
						<text class="weight">房租:</text>
						<input :class="priceOk ? 'noWarn' : 'warn'" v-model="rentInfo.price" @blur="checkPrice"/>元/月
					</view>
					<view class="cu-form-group fontSize">
						<text class="weight">交租时间:</text>
						<input :class="timeOk ? 'noWarn' : 'warn'" v-model="rentInfo.time" @blur="checkTime" :disabled="true" @click="showTimePicker"/>月
					</view>
					<view class="cu-form-group fontSize">
						<text class="weight">最少租期:</text>
						<input :class="leastTimeOk ? 'noWarn' : 'warn'" v-model="rentInfo.leastTime" @blur="checkLeastTime" :disabled="true" @click="showLeastTimePicker"/>月
					</view>
					<view class="cu-form-group fontSize">
						<text class="weight">基本押金:</text>
						<input :class="depositOk ? 'noWarn' : 'warn'" v-model="rentInfo.deposit" @blur="checkDeposit"/>元
					</view>
					<view class="cu-form-group fontSize">
						<text class="weight">钥匙押金:</text>
						<input :class="keydepositOk ? 'noWarn' : 'warn'" v-model="rentInfo.keyDeposit" @blur="checkKeyDeposit"/>元
					</view>
					<!-- !!!!! placeholder 在ios表现有偏移 建议使用 第一种样式 -->
					<view class="cu-form-group margin-top">
						<text class="title weight">输入介绍</text>
						<textarea :class="introductionOk ? 'noWarn' : 'warn'" maxlength="100" v-model="rentInfo.introduction" @blur="checkIntroduction" placeholder="限制100字以内"></textarea>
					</view>
					
					<!-- !!!!! placeholder 在ios表现有偏移 建议使用 第一种样式 -->
					<view class="cu-form-group margin-top" >
						<text class="title weight">输入规则</text>
						<textarea :class="RuleOk ? 'noWarn' : 'warn'" maxlength="100" v-model="rentInfo.rule" @blur="checkRule" placeholder="限制100字以内"></textarea>
					</view>
				</form>
			</view>
			<template v-slot:footer>
				<button class="cu-btn bg-blue flex justify-center" @click="modifyRent">去修改</button>
			</template>
		</uni-card>
		<uni-card title="封面信息" :is-shadow="true" mode="basic" note="true">
			<image :src="rentInfo.image" mode="aspectFit"></image>
			<template v-slot:footer>
				<button class="cu-btn bg-blue flex justify-center" @click="updateImage">修改图片</button>
			</template>
		</uni-card>
		<lb-picker ref="picker1" mode="selector" :list="times" @confirm="getTime"></lb-picker>
		<lb-picker ref="picker2" mode="selector" :list="leasttimes" @confirm="getLeasttime"></lb-picker>
	</view>
</template>

<script>
	var _this;
	import http from '../../common/http.js';
	import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
	export default {
		data() {
			return {
				rentInfo:{},
				access_token:'',
				userInfo:{},
				priceOk:true,
				depositOk:true,
				timeOk:true,
				leastTimeOk:true,
				TitleOk:true,
				introductionOk:true,
				RuleOk:true,
				keydepositOk:true,
				times:[
					{
						label:'一个月',
						value:1
					},
					{
						label:'两个月',
						value:2
					},
					{
						label:'三个月',
						value:3
					},
					{
						label:'六个月',
						value:6
					}
				],
				leasttimes:[
					{
						label:'一个月',
						value:1
					},
					{
						label:'二个月',
						value:2
					},
					{
						label:'三个月',
						value:3
					},
					{
						label:'四个月',
						value:4
					},
					{
						label:'五个月',
						value:5
					},
					{
						label:'六个月',
						value:6
					},
					{
						label:'七个月',
						value:7
					},
					{
						label:'八个月',
						value:8
					},
					{
						label:'九个月',
						value:9
					},
					{
						label:'十个月',
						value:10
					},
					{
						label:'十一个月',
						value:11
					},
					{
						label:'一年',
						value:12
					}
				]
			}
		},
		onLoad(e) {
			_this = this;
			this.rentInfo.hid = e.hid;
		},
		onShow() {
			this.access_token = uni.getStorageSync('access_token');
			this.userInfo = uni.getStorageSync('userInfo');
			this.getRent()
		},
		components:{
			uniCard
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
			checkFileRes(res){
				if (res.code === 500) {
					uni.showToast({
						icon: 'none',
						title: '服务器开小差了'
					});
				}
				if (res.code === 510) {
					uni.showToast({
						icon: 'none',
						title: res.data
					});
					setTimeout(function() {
						uni.navigateTo({
							url: '../login/reLogin?username=' + _this.userInfo.username
						});
					}, 1000);
				}
				if (res.code === 400) {
					uni.showToast({
						icon: 'none',
						title: '服务器开小差了'
					});
				}
			},
			checkPrice(){
				if(Number(this.rentInfo.price) <= 0 || this.rentInfo.price === '' || this.rentInfo.price.indexOf('.') != -1){
					this.priceOk = false;
				}else{
					this.priceOk = true;
				}
			},
			checkDeposit(){
				if(Number(this.rentInfo.deposit) <= 0 || this.rentInfo.deposit === '' || this.rentInfo.deposit.indexOf('.') != -1){
					this.depositOk = false;
				}else{
					this.depositOk = true;
				}
			},
			checkKeyDeposit(){
				if(Number(this.rentInfo.keyDeposit) <= 0 || this.rentInfo.keyDeposit === '' || this.rentInfo.keyDeposit.indexOf('.') != -1){
					this.keyDeposit = false;
				}else{
					this.keyDeposit = true;
				}
			},
			checkTime(){
				if(this.rentInfo.time === ''){
					this.timeOk = false;
				}else{
					this.timeOk = true;
				}
			},
			checkLeastTime(){
				if(this.rentInfo.leastTime === '' || Number(this.rentInfo.leastTime) < Number(this.rentInfo.time)){
					this.leastTimeOk = false;
				}else{
					this.leastTimeOk = true;
				}
			},
			checkTitle(){
				if(this.rentInfo.title.length == 0 || this.rentInfo.title.length >= 20){
					this.TitleOk = false;
					return false;
				}else{
					this.TitleOk = true;
					return true;
				}
			},
			checkIntroduction(){
				if(this.rentInfo.introduction === '' || this.rentInfo.introduction.length >= 100){
					this.introductionOk = false;
					return false;
				}else{
					this.introductionOk = true;
					return true;
				}
			},
			checkRule(){
				if(this.rentInfo.rule === '' || this.rentInfo.rule.length >= 100){
					this.RuleOk = false;
					return false;
				}else{
					this.RuleOk = true;
					return true;
				}
			},
			formCheck(){
				let titleCheck = this.checkTitle()
				let inCheck = this.checkIntroduction();
				let RuleCheck = this.checkRule()
				if(this.priceOk && this.depositOk && this.timeOk && this.leastTimeOk && this.keydepositOk && titleCheck && inCheck && RuleCheck){
					return true;
				}else{
					uni.showToast({
						title:"请修改红色部分内容",
						icon:"none"
					})
					return false;
				}
			},
			getRent(){
				uni.request({
					url:http.BaseUrl + 'rent/findRentByHid/'+_this.rentInfo.hid,
					method:"GET",
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					success: (res) => {
					    _this.checkResponse(res);
						if(res.data.code === 200){
							_this.rentInfo = res.data.data;
						}
					}
				})
			},
			updateImage(){
				uni.chooseImage({
					count:1,
					success: (res) => {
						uni.uploadFile({
							url:http.BaseUrl + 'rent/updateImg/'+_this.rentInfo.hid,
							header: {
								Authorization: 'bearer ' + _this.access_token
							},
							name:'file',
							filePath:res.tempFilePaths[0],
							success:(res)=>{
								let response = JSON.parse(res.data);
								console.log(response)
								if(response.code === 200){
									uni.showToast({
										title:"成功",
										icon:"success"
									})
									_this.getRent()
								}
							}
						})
					}
				})
				
			},
			showTimePicker(){
				this.$refs['picker1'].show();
			},
			getTime(e){
				this.rentInfo.time = e.value;
				this.rentInfo.leastTime = '';
			},
			showLeastTimePicker(){
				let time = this.rentInfo.time;
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
			getLeasttime(e){
				this.rentInfo.leastTime = e.value
			},
			modifyRent(){
				let ok = this.formCheck();
				if(ok){
					uni.request({
						url:http.BaseUrl + 'rent/putRent',
						method:"PUT",
						data:_this.rentInfo,
						header: {
							Authorization: 'bearer ' + _this.access_token
						},
						success: (res) => {
							_this.checkResponse(res);
							if(res.data.code === 200){
								uni.showToast({
									title:"修改成功",
									icon:"success"
								})
								uni.navigateBack({
									delta:1
								})
							}
						}
					})
				}else{
					uni.showToast({
						title: '请修改红色部分',
						icon: 'none'
					});
				}
			}
		}
	}
</script>

<style scoped>
.weight {
	font-weight: bold;
}
.left {
	margin-left: 20rpx;
}

.warn {
	border-bottom: 2rpx solid red;
}
.noWarn{
	border-bottom: 2rpx solid black;
}
.fontSize{
	font-size: 35rpx;
}
</style>
