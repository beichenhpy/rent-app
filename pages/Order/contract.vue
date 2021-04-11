<template>
	<view>
		<uni-card title="合同信息" :is-shadow="true" note="true">
			<view class="flex justify-between" v-if="confirmOk">
				<text class="text-black">租客是否签合同:</text>
				<text class="text-red">{{ order.renterConfirm === 1 ? '已签' : '未签' }}</text>
			</view>
			<view class="flex justify-between" v-if="confirmOk">
				<text class="text-black">房东是否签合同:</text>
				<text class="text-red">{{ order.ownerConfirm === 1 ? '已签' : '未签' }}</text>
			</view>
			<view class="flex justify-between">
				<text class="text-black">合同保存路径</text>
				<view class="text-red" @tap="openWord">{{ saveFile === '' ? '无合同' : '点击打开' }}</view>
			</view>
			<view>{{saveFile}}</view>
			<template v-slot:footer>
				<view class="flex justify-center">
					<button v-if="confirmOk" class="cu-btn round bg-blue" @click="sendECard">去上传电子签名</button>
					<button v-if="confirmOk" class="cu-btn round bg-green" style="margin-left: 10rpx;" @click="signContract">签合同</button>
					<button class="cu-btn round bg-green" style="margin-left: 10rpx;" @click="download">下载合同</button>
				</view>
			</template>
		</uni-card>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
export default {
	data() {
		return {
			oid: '',
			type: '',
			access_token: '',
			userInfo: {},
			order:{},
			contract: '',
			isFinish: '',
			saveFile:'',
			noPay: '',
			confirmOk:false
		};
	},
	onLoad(e) {
		_this = this;
		this.oid = e.oid;
		this.type = e.type;
		this.isFinish = e.isFinish;
	},
	components: {
		uniCard
	},
	onShow() {
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
		this.contract = uni.getStorageSync('contract' + this.oid);
		if (this.isFinish == 0) {
			this.getOrder();
			this.getNoPay();
		}
		let saveFile = uni.getStorageSync('contract' + _this.oid)
		if(saveFile.length != 0){
			this.saveFile = saveFile;
		}
	},
	onPullDownRefresh() {
		if (this.isFinish == 0) {
			this.getOrder();
			this.getNoPay();
		}
		let saveFile = uni.getStorageSync('contract' + _this.oid)
		if(saveFile.length != 0){
			this.saveFile = saveFile;
		}
		uni.stopPullDownRefresh();
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
		getNoPay() {
			uni.request({
				url: http.BaseUrl + 'order/findNoPay/' + _this.oid,
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.noPay = res.data.data;
					}
				}
			});
		},
		getOrder() {
			uni.request({
				url: http.BaseUrl + 'order/findOrder',
				method: 'GET',
				data:{
					type:_this.type,
					oid:_this.oid
				},
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.order = res.data.data;
						if(_this.order.renterConfirm === 0 || _this.order.ownerConfirm === 0){
							_this.confirmOk = true;
						}
					}
				}
			});
		},
		sendECard() {
			if (this.noPay != 0) {
				uni.showToast({
					title: '有账单未确认支付',
					icon: 'none'
				});
			} else {
				let ok = this.checkConfirm();
				if (ok) {
					uni.navigateTo({
						url: '../eCard/eCard'
					});
				}
			}
			
		},
		checkConfirm() {
			if (_this.type === 'owner') {
				if (_this.order.renterConfirm === 0) {
					uni.showToast({
						title: '您还不能签合同，由于租客还没签，请联系租客',
						icon: 'none'
					});
					return false;
				} else if (_this.order.ownerConfirm === 1) {
					uni.showToast({
						title: '您已经签署过合同了',
						icon: 'none'
					});
					return false;
				} else {
					return true;
				}
			} else {
				if (_this.order.renterConfirm === 0) {
					return true;
				} else {
					uni.showToast({
						title: '您已经签署过合同了',
						icon: 'none'
					});
					return false;
				}
			}
		},
		signContract() {
			if (this.noPay != 0) {
				uni.showToast({
					title: '有账单未确认支付',
					icon: 'none'
				});
			} else {
				let ok = this.checkConfirm();
				if (ok) {
					uni.request({
						url: http.BaseUrl + 'user/findECard',
						method: 'GET',
						header: {
							Authorization: 'bearer ' + _this.access_token
						},
						success: res => {
							_this.checkResponse(res);
							if (res.data.code === 200) {
								if (res.data.data === 0) {
									uni.showToast({
										title: '您未上传电子签名',
										icon: 'none'
									});
								} else {
									uni.showLoading({
										title: '正在签署'
									});
									uni.request({
										url: http.BaseUrl + 'order/updateContract/'+_this.type+'?oid='+_this.oid,
										method: 'PUT',
										header: {
											Authorization: 'bearer ' + _this.access_token
										},
										complete: res => {
											uni.hideLoading();
											_this.checkResponse(res);
											if (res.data.code === 200) {
												uni.showToast({
													title: '签署成功',
													icon: 'success'
												});
												_this.getOrder();
											}else{
												uni.showToast({
													icon: 'none',
													title: '服务器开小差了'
												});
											}
										}
									});
								}
							}
						}
					});
				}
			}
		},
		downLoadFile() {
			uni.showLoading({
				title: '正在下载'
			});
			uni.request({
				url: http.BaseUrl + 'order/downloadContract/' + _this.oid,
				data: {
					type: _this.type
				},
				complete: res => {
					uni.hideLoading()
					_this.checkResponse(res);
					if (res.data.code === 200) {
						if(res.data.data != 'https://files.beichenhpy.cn/null'){
							uni.downloadFile({
								url: res.data.data,
								success: res => {
									uni.saveFile({
										tempFilePath: res.tempFilePath,
										success: resp => {
											console.log(resp.savedFilePath);
											_this.saveFile = resp.savedFilePath;
											uni.setStorageSync('contract' + _this.oid, resp.savedFilePath);
											uni.showToast({
												title:"文件保存在:"+_this.saveFile,
												icon:"none"
											})
										}
									});
								}
							});
						}
					}
				}
			});
		},
		download() {
			if (this.isFinish == 0) {
				if (_this.order.renterConfirm === 1 && _this.order.ownerConfirm === 1) {
					this.downLoadFile();
				} else {
					uni.showToast({
						title: '合同还未完成',
						icon: 'none'
					});
				}
			} else {
				this.downLoadFile();
			}
		},
		openWord() {
			uni.openDocument({
				filePath: _this.saveFile
			});
		}
	}
};
</script>

<style></style>
