<template>
	<view style="margin-bottom: 20%;">
		<view v-for="(items, index) in bills" :key="index">
			<uni-card title="账单信息" :is-shadow="true" :extra="items.createTime" mode="basic">
				<view class="flex justify-between" style="margin-left: 30rpx;">
					<text class="text-black weight">金额:<text style="margin-left: 20rpx; color: red;">{{items.price}}元</text></text>
					<view>
						<button class="cu-btn bg-blue round" @click="getPriceInfo(items.priceInfo)">查看金额</button>
					</view>
				</view>
				<view class="flex justify-between margin">
					<text class="text-black weight">租客确认情况</text>
					<text class="text-red weight">{{items.renterCheck === 1?'已确认':'未确认'}}</text>
				</view>
				<view class="flex justify-between margin">
					<text class="text-black weight">房东确认情况</text>
				    <text class="text-red weight">{{items.ownerCheck === 1?'已确认':'未确认'}}</text>
				</view>
				<view class="flex justify-center">
					<button class="cu-btn bg-green round" @click="confirmOk(items)">确认支付完成</button>
					<button v-if="type == 'renter'" class="cu-btn bg-blue round" @click="printReceipt(items)">下载收据</button>
					<button v-if="type == 'renter'" class="cu-btn bg-blue round" @click="showReceipt(items)">查看收据</button>
				</view>
			</uni-card>
		</view>
		<view>
			<button v-if="type =='owner' && isFinish == 0" class="bg-blue bottomBtn" @click="openAddModal" >添加房租</button>
		</view>
		<price-info-modal :priceModal="priceInfoModal" :priceInfo="priceInfo" @closeModal = "closeModal"></price-info-modal>
		<add-bill-modal :oid="oid" :addBillModal="addBillModal" @closeModal = "closeModal" @addBill="addBill"></add-bill-modal>
	</view>
</template>

<script>
	var _this;
	import http from '../../common/http.js'
	import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
	import priceInfoModal from '../../components/order/priceModal.vue'
	import addBillModal from '../../components/order/addBillModal.vue'
	export default {
		data() {
			return {
				bills:[],
				oid:'',
				hid:'',
				type:'',
				isFinish:0,
				access_token: '',
				userInfo: {},
				priceInfo:{},
				priceInfoModal:false,
				addBillModal:false,
				billTime:''
			}
		},
		onLoad(e) {
			_this = this;
			this.oid = e.oid;
			this.type = e.type;
			this.hid = e.hid;
			this.billTime = e.billTime;
			this.isFinish = e.isFinish;
		},
		components:{
			uniCard,
			priceInfoModal,
			addBillModal
		},
		onShow() {
			this.access_token = uni.getStorageSync('access_token');
			this.userInfo = uni.getStorageSync('userInfo');
			this.getBills();
		},
		onPullDownRefresh() {
			this.getBills();
			uni.stopPullDownRefresh()
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
			getBills(){
				uni.request({
					url:http.BaseUrl + 'order/findAllBill/'+_this.oid,
					method:"GET",
					data:{
						type:_this.type
					},
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					success: (res) => {
						_this.checkResponse(res);
						if(res.data.code === 200){
							_this.bills = res.data.data;
						}
					}
				})
			},
			checkConfirm(items){
				if(this.type === 'renter'){
					if(items.renterCheck === 1){
						uni.showToast({
							title:"您已确认",
							icon:"none"
						})
						return false;
					}
					return true;
				}
				if(this.type === 'owner'){
					if(items.ownerCheck === 1){
						uni.showToast({
							title:"您已确认",
							icon:"none"
						})
						return false;
					}
					return true;
				}
			},
			confirmOk(items){
				let ok = this.checkConfirm(items);
				if(ok){
					uni.showLoading({
						title:"正在确认请稍后"
					})
					if(this.type === 'renter'){
						uni.request({
							url:http.BaseUrl + 'order/renterCheck',
							method:"PUT",
							data:{
								bid:items.bid,
								oid:items.oid
							},
							header: {
								Authorization: 'bearer ' + _this.access_token
							},
							complete: (res) => {
								uni.hideLoading()
								_this.checkResponse(res);
								if(res.data.code === 200){
									uni.showToast({
										title:"成功",
										icon:"success"
									})
									this.getBills();
								}
							}
						})
					}else{
						if(items.renterCheck == 1){
							uni.request({
								url:http.BaseUrl + 'order/ownerCheck',
								method:"PUT",
								data:{
									bid:items.bid,
									oid:items.oid
								},
								header: {
									Authorization: 'bearer ' + _this.access_token
								},
								complete: (res) => {
									uni.hideLoading()
									_this.checkResponse(res);
									if(res.data.code === 200){
										uni.showToast({
											title:"成功",
											icon:"success"
										})
										this.getBills();
									}
								}
							})
						}else{
							uni.showToast({
								title:"等待租客确认支付",
								icon:"none"
							})
						}
					}
				}	
				
			},
			getPriceInfo(priceInfo){
				this.priceInfo = priceInfo;
				this.priceInfoModal = true;
			},
			openAddModal(){
				let length = _this.bills.length;
				let lastTime = _this.bills[length-1].createTime;
				uni.request({
					url:http.BaseUrl + 'order/checkAddBill',
					data:{
						lastTime:lastTime,
						billTime:_this.billTime
					},
					method:"GET",
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					success: (res) => {
						_this.checkResponse(res);
						if(res.data.code === 200){
							if(res.data.data){
								this.addBillModal = true;
							}else{
								uni.showToast({
									title:"还没到交租时期，不能添加",
									icon:"none"
								})
							}
						}
					}
				})
			},
			closeModal(e){
				if(e){
					this.priceInfoModal = false;
					this.addBillModal = false;
				}
			},
			addBill(e){
				if(e.elect === '' || e.electCar ==='' || e.water === ''){
					uni.showToast({
						title:"您有项目未填",
						icon:"none"
					})
				}else if(e.elect.indexOf(".") != -1 || e.electCar.indexOf(".") != -1 || e.water.indexOf(".") != -1){
					uni.showToast({
						title:"不能输入小数",
						icon:"none"
					})
				}else{
					uni.showLoading({
						title:"正在添加"
					})
					uni.request({
						url:http.BaseUrl + 'order/addPriceInfo/'+_this.oid,
						method:"POST",
						data:{
							carRecord:e.electCar,
							waterRecord:e.water,
							houseRecord:e.elect
						},
						header: {
							Authorization: 'bearer ' + _this.access_token
						},
						complete: (res) => {
							uni.hideLoading()
							_this.checkResponse(res);
							if(res.data.code === 200){
								uni.showToast({
									title:res.data.data,
									icon:"success"
								})
								_this.getBills();
								_this.addBillModal = false;
							}
						}
					})
				}
			},
			printReceipt(items){
				uni.showLoading({
					title:"正在下载"
				})
				uni.request({
					url:http.BaseUrl + 'file/getReceipt',
					data:{
						uid:_this.userInfo.uid,
						bid:items.bid
					},
					complete: (res) => {
						uni.hideLoading()
						_this.checkResponse(res);
						if(res.data.code === 200){
							uni.downloadFile({
								url:res.data.data,
								success: (res) => {
									uni.saveFile({
										tempFilePath:res.tempFilePath,
										success: (res) => {
											uni.setStorageSync(items.bid,res.savedFilePath);
											uni.showToast({
												title:"下载完成，保存在:"+res.savedFilePath,
												icon:"none"
											})
											setTimeout(function(){
												uni.hideLoading()
											},1000)
										}
									})
								}
							})
						}
					}
				})
			},
			showReceipt(items){
				let path = uni.getStorageSync(items.bid);
				if(path.length != 0){
					uni.openDocument({
						filePath:path
					})
				}else{
					uni.showToast({
						title:"请重新下载",
						icon:"none"
					})
				}
				
			}
		}
	}
</script>

<style scoped>
.weight{
	font-weight: bold;
}
.margin{
	margin-top: 20rpx;
}
.bottomBtn{
	z-index: 999;
	position: fixed;
	bottom: 0%;
	width: 100%;
}
</style>
