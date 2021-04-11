<template>
	<view>
		<view class="cu-bar bg-white margin-top">
			<view class="action">请选择身份证正面照片</view>
			<view class="action">{{ imgList.length }}/1</view>
		</view>
		<view class="cu-form-group">
			<view class="grid col-1 grid-square flex-sub">
				<view class="bg-img" v-for="(item, index) in imgList" :key="index" @tap="ViewImage" :data-url="imgList[index]">
					<image :src="imgList[index]" mode="aspectFill"></image>
					<view class="cu-tag bg-red" @tap.stop="DelImg" :data-index="index"><text class="cuIcon-close"></text></view>
				</view>
				<view class="solids" @tap="ChooseImage" v-if="imgList.length < 1"><text class="cuIcon-cameraadd"></text></view>
			</view>
		</view>
		<view><button @click="author">认证</button></view>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
export default {
	data() {
		return {
			imgList: [],
			access_token: '',
			userInfo: {}
		};
	},
	onLoad(value) {
		_this = this;
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
	},
	onBackPress(e) {
		if (e.from === 'backbutton') {
			uni.navigateBack({
				delta: 1
			});
			return true;
		}
	},
	methods: {
		checkResponse(res) {
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
					title: "该认证信息已经被使用"
				});
			}
		},
		ChooseImage() {
			uni.chooseImage({
				count: 1, //默认9
				sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
				sourceType: ['album'], //从相册选择
				success: res => {
					if (this.imgList.length != 0) {
						this.imgList = this.imgList.concat(res.tempFilePaths);
					} else {
						this.imgList = res.tempFilePaths;
					}
				}
			});
		},
		ViewImage(e) {
			uni.previewImage({
				urls: this.imgList,
				current: e.currentTarget.dataset.url
			});
		},
		DelImg(e) {
			uni.showModal({
				title: '警告',
				content: '确定要删除吗？',
				cancelText: '取消',
				confirmText: '确定',
				success: res => {
					if (res.confirm) {
						this.imgList.splice(e.currentTarget.dataset.index, 1);
					}
				}
			});
		},
		author() {
			if (this.imgList.length != 0) {
				uni.showLoading({
					title:"正在认证请稍后"
				})
				uni.uploadFile({
					url: http.BaseUrl + 'user/addIdCardInfoForRegister',
					filePath: _this.imgList[0],
					name: 'file',
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					complete: function(response) {
						uni.hideLoading()
						let res = JSON.parse(response.data);
						_this.checkResponse(res);
						if (res.code === 200) {
							uni.showToast({
								icon: 'success',
								title: '认证成功'
							});
							setTimeout(function(){
								uni.switchTab({
									url:"../User/User"
								})
							},500)
						}
					}
				});
			} else {
				uni.showToast({
					title: '请选择身份证正面照片',
					icon: 'none'
				});
			}
		}
	}
};
</script>

<style></style>
