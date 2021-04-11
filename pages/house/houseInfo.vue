<template>
	<view>
		<!-- 房产证图片 -->
		<uni-card :is-shadow="true" title="房产证信息" note="true">
			<view class="text-grey flex justify-center">
				<image v-if="houseInfo.houseCard != null" :src="houseInfo.houseCard" @click="showImage"></image>
				<text v-else>未上传房产证</text>
			</view>
			<view class="flex justify-between" style="margin-top: 30rpx;">
				<view>
					<text style="font-weight: bold;">房产证号:</text>
					<text class="text-grey">{{ houseInfo.houseCardNum === null ? '无数据' : houseInfo.houseCardNum }}</text>
				</view>
				<view><button class="cu-btn bg-yellow" @click="showUpdateHouseCardNum">修改</button></view>
			</view>
			<template v-slot:footer>
				<button class="cu-btn bg-green flex justify-center round" @click="uploadHouseCard">上传房产证</button>
			</template>
		</uni-card>
		<!-- 具体信息 -->
		<uni-card :is-shadow="true" title="设施信息" note="true">
			<view class="flex justify-between">
				<text style="font-weight: bold;">床</text>
				<view v-if="houseInfo.isBed === 1"><text class="cuIcon-check bg-green"></text></view>
				<view v-else><text class="cuIcon-close bg-red"></text></view>
			</view>
			<view class="flex justify-between">
				<text style="font-weight: bold;">洗衣机</text>
				<view v-if="houseInfo.isWash === 1"><text class="cuIcon-check bg-green"></text></view>
				<view v-else><text class="cuIcon-close bg-red"></text></view>
			</view>
			<view class="flex justify-between">
				<text style="font-weight: bold;">衣柜</text>
				<view v-if="houseInfo.isRobe === 1"><text class="cuIcon-check bg-green"></text></view>
				<view v-else><text class="cuIcon-close bg-red"></text></view>
			</view>
			<view class="flex justify-between">
				<text style="font-weight: bold;">淋浴</text>
				<view v-if="houseInfo.isWarm === 1"><text class="cuIcon-check bg-green"></text></view>
				<view v-else><text class="cuIcon-close bg-red"></text></view>
			</view>
			<view class="flex justify-between">
				<text style="font-weight: bold;">空调</text>
				<view v-if="houseInfo.isAir === 1"><text class="cuIcon-check bg-green"></text></view>
				<view v-else><text class="cuIcon-close bg-red"></text></view>
			</view>
			<view class="flex justify-between">
				<text style="font-weight: bold;">冰箱</text>
				<view v-if="houseInfo.isFridge === 1"><text class="cuIcon-check bg-green"></text></view>
				<view v-else><text class="cuIcon-close bg-red"></text></view>
			</view>
			<template v-slot:footer>
				<button class="cu-btn bg-green flex justify-center round" @click="uploadInfo">修改配置</button>
			</template>
		</uni-card>
		<uni-card :is-shadow="true" title="水电表读数" note="true">
			<view class="flex justify-between">
				<text class="text-black">房屋水表读数:</text>
				<text class="text-grey">{{houseInfo.waterRecord == null ? '未输入' : houseInfo.waterRecord}}吨</text>
			</view>
			<view class="flex justify-between">
				<text class="text-black">电车电表读数:</text>
				<text class="text-grey">{{houseInfo.carRecord == null ? '未输入' : houseInfo.carRecord}}度</text>
			</view>
			<view class="flex justify-between">
				<text class="text-black">房屋电表读数:</text>
				<text class="text-grey">{{houseInfo.houseRecord == null ? '未输入' : houseInfo.houseRecord}}度</text>
			</view>
			<template v-slot:footer>
				<button class="cu-btn bg-green flex justify-center round" @click="openRecord">输入水电表读数</button>
			</template>
		</uni-card>
		<uni-card :is-shadow="true" title="图片视频" note="true">
			<view class="flex justify-between">
				<button class="cu-btn round" @click="showImages">查看图片</button>
				<button class="cu-btn round" @click="showVideo">查看视频</button>
			</view>
			<template v-slot:footer>
				<view class="flex justify-between">
					<button class="cu-btn bg-green round" @click="uploadImages">上传图片</button>
					<button class="cu-btn bg-green round" @click="uploadVideo">上传视频</button>
				</view>
			</template>
		</uni-card>
		<!-- 修改房产证号modal -->
		<house-card-num-modal :Modal="updateHouseCardModal" :hid="houseInfo.hid" @updateHouseNum="updateHouseCardNum"></house-card-num-modal>
		<!-- 修改修改读数modal -->
		<view class="cu-modal" :class="RecordModal ? 'show' : ''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end"><view class="content">输入水电表读数</view></view>
				<view class="padding-xl">
					<view class="cu-form-group margin-top" :class="waterOk ? '' : 'warn'">
						<view class="title">房屋水表读数</view>
						<input placeholder="输入整数" type="number" maxlength="4" v-model="houseRecordQuery.waterRecord" @blur="checkWater"/>吨
					</view>
					<view class="cu-form-group margin-top" :class="carOk ? '' : 'warn'">
						<view class="title">电车电表读数</view>
						<input placeholder="输入整数" type="number" maxlength="6" v-model="houseRecordQuery.carRecord" @blur="checkCar"/>度
					</view>
					<view class="cu-form-group margin-top" :class="houseOk ? '' : 'warn'">
						<view class="title">房屋电表读数</view>
						<input placeholder="输入整数" type="number" maxlength="6" v-model="houseRecordQuery.houseRecord" @blur="checkHouse"/>度
					</view>
				</view>
				<view class="cu-bar bg-white">
					<view class="action margin-0 flex-sub text-green solid-left" @tap="closeRecord">取消</view>
					<view class="action margin-0 flex-sub  solid-left" @tap="updateRecord">确定</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
import houseCardNumModal from '../../components/house/updateHouseNum.vue';
export default {
	data() {
		return {
			houseInfo: {},
			access_token: '',
			userInfo: {},
			isRented: '',
			isOnRent: '',
			updateHouseCardModal: false,
			imageList:[],
			RecordModal:false,
			houseRecordQuery:{
				waterRecord:'',
				houseRecord:'',
				carRecord:'',
				hid:'',
				houseInfoId:''
			},
			waterOk:true,
			carOk:true,
			houseOk:true
		};
	},
	onLoad(e) {
		_this = this;
		this.houseInfo.hid = e.hid;
		this.isRented = e.isRented;
		this.isOnRent = e.isOnRent;
	},
	onShow() {
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
		this.getHouseInfo();
	},
	components: {
		uniCard,
		houseCardNumModal
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
		checkFileResponse(res) {
			if (res.code === 500) {
				uni.showToast({
					icon: 'none',
					title: '服务器开小差了'
				});
			}
			if (res.code === 510) {
				uni.showToast({
					icon: 'none',
					title: '登录身份过期'
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
					title: '服务器异常'
				});
			}
		},
		getHouseInfo() {
			uni.request({
				url: http.BaseUrl + 'house/findHouseInfo/' + _this.houseInfo.hid,
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: function(res) {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.houseInfo = res.data.data;
						if(res.data.data != null){
							_this.imageList = res.data.data.images;
						}
					}
				}
			});
		},
		checkRent() {
			if (this.isRented === '1') {
				uni.showToast({
					title: '已出租不能修改',
					icon: 'none'
				});
				return false;
			} else if(this.isOnRent === '1'){
				uni.showToast({
					title: '已发布不能修改',
					icon: 'none'
				});
				return false;
			}else{
				return true;
			}
		},
		uploadHouseCard() {
			let ok = this.checkRent();
			if (ok) {
				uni.chooseImage({
					count: 1,
					success: function(file) {
						uni.showLoading({
							title: '正在上传'
						});
						uni.uploadFile({
							url: http.BaseUrl + 'house/updateHouseCard/' + _this.houseInfo.houseInfoId,
							filePath: file.tempFilePaths[0],
							name: 'file',
							header: {
								Authorization: 'bearer ' + _this.access_token
							},
							success: function(response) {
								uni.hideLoading();
								let res = JSON.parse(response.data);
								_this.checkFileResponse(res);
								if (res.code === 200) {
									_this.getHouseInfo();
									setTimeout(function() {
										uni.showToast({
											title: '上传成功',
											icon: 'success'
										});
									}, 1000);
								}
							}
						});
						uni.hideLoading();
					}
				});
			}
		},
		showImage() {
			let imagesList = [];
			imagesList.push(_this.houseInfo.houseCard);
			uni.previewImage({
				urls: imagesList
			});
		},
		showUpdateHouseCardNum() {
			let ok = this.checkRent();
			if (ok) {
				this.updateHouseCardModal = true;
			}
		},
		updateHouseCardNum(ok) {
			if (ok === 'close') {
				this.updateHouseCardModal = false;
			} else if (ok === 'updateNo') {
				this.updateHouseCardModal = true;
			} else {
				this.updateHouseCardModal = false;
				this.getHouseInfo();
				uni.showToast({
					title: '修改成功',
					icon: 'success'
				});
			}
		},
		uploadInfo() {
			let ok = this.checkRent();
			if (ok) {
				uni.navigateTo({
					url: 'modifyhouseInfo?hid=' + _this.houseInfo.hid+"&houseInfoId="+_this.houseInfo.houseInfoId
				});
			}
		},
		showImages() {
			if (_this.imageList !== null) {
				uni.previewImage({
					urls: _this.imageList
				});
			} else {
				uni.showToast({
					title: '还没有图片信息',
					icon: 'none'
				});
			}
		},
		showVideo() {
			let video = this.houseInfo.video;
			if (video === null || video === '') {
				uni.showToast({
					title: '还没有视频信息',
					icon: 'none'
				});
			} else {
				uni.navigateTo({
					url: '../rentInfo/video?video=' + video
				});
			}
		},
		uploadImages() {
			let ok = this.checkRent();
			if (ok) {
				uni.chooseImage({
					count: 3,
					sourceType: ['album'],
					success: function(file) {
						let imageList = [];
						imageList = imageList.concat(file.tempFilePaths);
						let files = imageList.map((value, index) => {
							return {
								name: 'file' + index,
								uri: value
							};
						});
						uni.showLoading({
							title:"正在上传"
						})
						uni.uploadFile({
							url: http.BaseUrl + 'house/updateHouseImage/' + _this.houseInfo.houseInfoId,
							files: files,
							header: {
								Authorization: 'bearer ' + _this.access_token
							},
							success: function(response) {
								uni.hideLoading();
								let res = JSON.parse(response.data);
								_this.checkFileResponse(res);
								if (res.code === 200) {
									_this.getHouseInfo();
									setTimeout(function() {
										uni.showToast({
											title: '上传成功',
											icon: 'success'
										});
									}, 1000);
								}
							}
						});
					}
				});
			}
		},
		uploadVideo() {
			let ok = this.checkRent();
			if (ok) {
				uni.chooseVideo({
					camera:'back',
					success: res => {
						console.log(res);
						if (res.size > 1024 * 1024 * 10) {
							uni.showToast({
								title: '视频大小不能超过10M',
								icon: 'none'
							});
						} else {
							uni.showLoading({
								title:"正在上传"
							})
							uni.uploadFile({
								url: http.BaseUrl + 'house/updateHouseVideo/' + _this.houseInfo.houseInfoId,
								name: 'file',
								header: {
									Authorization: 'bearer ' + _this.access_token
								},
								filePath: res.tempFilePath,
								success: res => {
									uni.hideLoading()
									let response = JSON.parse(res.data);
									_this.checkFileResponse(response);
									if ((response.code = 200)) {
										uni.showToast({
											title: '上传成功',
											icon: 'success'
										});
									}
									_this.getHouseInfo();
								}
							});
						}
					}
				});
			}
		},
		closeRecord(){
			this.RecordModal = false;
			this.houseRecordQuery  ={
				waterRecord:'',
				houseRecord:'',
				carRecord:'',
				hid:'',
				houseInfoId:''
			};
			this.waterOk = true;
			this.carOk = true;
			this.houseOk = true;
		},
		openRecord(){
			if(this.isRented == 1){
				uni.showToast({
					title:"已经出租不能修改",
					icon:"none"
				})
			}else{
				if(this.isOnRent == 1){
					uni.showToast({
						title:"已经发布不能修改，请先下架",
						icon:"none"
					})
				}else{
					this.RecordModal = true;
				}
			}
		},
		checkWater(){
			if(Number(this.houseRecordQuery.waterRecord) < 0 || this.houseRecordQuery.waterRecord == '' || !/^\d+$/.test(this.houseRecordQuery.waterRecord)){
				this.waterOk = false;
				return false
			}else{
				this.waterOk = true;
				return true;
			}
		},
		checkCar(){
			if(Number(this.houseRecordQuery.carRecord) < 0 || this.houseRecordQuery.carRecord == '' || !/^\d+$/.test(this.houseRecordQuery.carRecord)){
				this.carOk = false;
				return false
			}else{
				this.carOk = true;
				return true;
			}
		},
		checkHouse(){
			if(Number(this.houseRecordQuery.houseRecord) < 0 || this.houseRecordQuery.houseRecord == '' || !/^\d+$/.test(this.houseRecordQuery.houseRecord)){
				this.houseOk = false;
				return false
			}else{
				this.houseOk = true;
				return true;
			}
		},
		updateRecord(){
			if(this.checkCar() && this.checkHouse() && this.checkWater()){
				this.houseRecordQuery.hid = this.houseInfo.hid;
				this.houseRecordQuery.houseInfoId = this.houseInfo.houseInfoId;
				uni.showLoading({
					title:"正在修改"
				})
				uni.request({
					url:http.BaseUrl + 'house/updateHouseRecord',
					method:"PUT",
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					data:_this.houseRecordQuery,
					success: (res) => {
						uni.hideLoading();
						_this.checkResponse(res);
						if(res.data.code === 200){
							uni.showToast({
								title:"修改成功",
								icon:"success"
							})
							_this.RecordModal = false;
							_this.getHouseInfo();
						}
					}
				})
			}else{
				uni.showToast({
					title:"填写格式错误，应为非负整数且不能为空",
					icon:"none"
				})
			}
			
		}
	}
};
</script>

<style scoped>
	.warn {
		border-bottom: 2rpx solid red;
	}
</style>
