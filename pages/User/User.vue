<template>
	<view style="margin-bottom: 20%;">
		<!--主体-->
		<view class="meMain">
			<view class="mePosition">
				<view class="meMainBox">
					<!--头部-->
					<view class="meHead">
						<view class="meHeadAvatar"><image :src="avatarUrl" mode="aspectFill" @click="OpenModal('profilePhoto')"></image></view>
						<view class="meHeadName">
							<view @tap="OpenModal('nickName')">{{ userInfo.nickName }}</view>
						</view>
						<view class="meHeadName" :class="userInfo.success === 1 ? 'success' : 'unsuccess'">
							<view @click="toAuthorization">{{ userInfo.success === 1 ? '已认证' : '去认证' }}</view>
						</view>
					</view>
					<!-- 菜单 -->
					<user-list :sex="userInfo.sex" @changeSex="changeSex"></user-list>
					<!-- 修改昵称 -->
					<update-nick-name :nickModal="nickModal" @updateNickName="updateNickName"></update-nick-name>
					<!-- 修改头像 -->
					<view class="cu-modal" :class="profilePhotoModal ? 'show' : ''">
						<view class="cu-dialog">
							<view class="cu-bar bg-white justify-end">
								<view class="content">点击图片修改头像</view>
								<view class="action" @tap="profilePhotoModal = false"><text class="cuIcon-close text-red"></text></view>
							</view>
							<view class="padding-xl">
								<view class="margin-top">
									<view class="image"><image :src="newPhoto" @click="choosePhoto"></image></view>
								</view>
							</view>
							<view class="cu-bar bg-white">
								<view class="action margin-0 flex-sub text-green solid-left" @tap="profilePhotoModal = false">取消</view>
								<view class="action margin-0 flex-sub  solid-left" @tap="upload">确定</view>
							</view>
						</view>
					</view>
				</view>
			</view>
			<view class="padding flex flex-direction">
			<button class="cu-btn bg-red margin-tb-sm lg" style=" position: fixed; bottom: 0%; left: 35%;" @click="exit">退出登录</button>
			</view>
		</view>
		<!--主体-->
	</view>
</template>

<script>
var _this;
import updateNickName from '../../components/updateUser/updateNickname.vue';
import userList from '../../components/userList/userList.vue';
import http from '../../common/http.js';

export default {
	data() {
		return {
			avatarUrl: '../../static/avatar.png',
			userInfo: {
				nickName: '未登录',
				success: ''
			},
			profilePhoto: '',
			newPhoto: '',
			nickModal: false,
			profilePhotoModal: false,
			login:false
		};
	},
	components: {
		userList,
		updateNickName
	},
	onShow() {
		this.getUserInfo()
	},
	onLoad() {
		_this = this;
	},
	onPullDownRefresh() {
		this.getUserInfo();
		uni.stopPullDownRefresh();
		if(this.login){
			uni.showToast({
				title: '获取到最新信息',
				icon: 'success'
			});
		}else{
			uni.showToast({
				title: '尚未登录',
				icon: 'none'
			});
		}
	},
	methods: {
		checkToken(access_token) {
			if (access_token.length !== 0) {
				return true;
			} else {
				return false;
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
				let userInfo = uni.getStorageSync('userInfo');
				uni.showToast({
					icon: 'none',
					title: res.data.data
				});
				setTimeout(function() {
					uni.navigateTo({
						url: '../login/reLogin?username=' + userInfo.username
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
		checkFileResponse(res){
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
					title: res.data
				});
			}
		},
		getUserInfo() {
			let access_token = uni.getStorageSync('access_token');
			let ok = this.checkToken(access_token);
			if (ok) {
				uni.request({
					url: http.BaseUrl + 'user/findMyInfo',
					header: {
						Authorization: 'bearer ' + access_token
					},
					method: 'GET',
					success: function(res) {
						_this.checkResponse(res);
						if (res.data.code === 200) {
							_this.userInfo = res.data.data;
							_this.avatarUrl = res.data.data.profilePhoto;
							_this.newPhoto = _this.avatarUrl;
							_this.login = true;
							uni.setStorageSync('userInfo', res.data.data);
						}
					}
				});
			}
		},
		OpenModal(type) {
			let access_token = uni.getStorageSync('access_token');
			let ok  = this.checkToken(access_token);
			if(ok){
				if (type === 'nickName') {
					this.nickModal = true;
				} else if (type === 'profilePhoto') {
					this.profilePhoto = this.avatarUrl;
					this.profilePhotoModal = true;
				}
			}else{
				uni.navigateTo({
					url:"../login/login?to=1"
				})
			}
		},
		updateNickName(ok) {
			if (ok === 'close') {
				this.nickModal = false;
			} else if (ok === 'updateNo') {
				this.nickModal = true;
			} else {
				this.nickModal = false;
				this.getUserInfo();
			}
		},
		choosePhoto() {
			uni.chooseImage({
				count: 1,
				success: function(file) {
					_this.newPhoto = file.tempFilePaths[0];
				}
			});
		},
		upload() {
			let access_token = uni.getStorageSync('access_token');
			let ok = this.checkToken(access_token);
			if(ok){
				uni.uploadFile({
					url: http.BaseUrl + 'user/updateProfilePhoto',
					filePath: _this.newPhoto,
					name:"file",
					header: {
						Authorization: 'bearer ' + access_token
					},
					success: function(response) {
						let res = JSON.parse(response.data)
						_this.checkFileResponse(res);
						if(res.code === 200){
							_this.profilePhotoModal  = false;
							_this.getUserInfo();
						}
					}
				});
			}
		},
		exit(){
			uni.clearStorageSync();
			if(this.login){
				this.avatarUrl = '../../static/avatar.png';
				this.userInfo = {
					nickName: '未登录',
					success: ''
				}
				uni.showToast({
					title:"成功退出",
					icon:"success"
				});
				this.login = false;
			}else{
				uni.showToast({
					title:"您尚未登录",
					icon:"none"
				})
			}
		},
		toAuthorization(){
			if(this.login){
				if(_this.userInfo.success === 0){
					uni.navigateTo({
						url:'../Authorization/Authorization?to=1'
					})
				}else{
					uni.showToast({
						title:"您已经通过实名认证",
						icon:"none"
					})
				}
			}else{
				uni.showToast({
					title:"请先去登录",
					icon:"none"
				})
			}
			
		},
		changeSex(e){
			if(e){
				uni.showToast({
					title:"修改成功",
					icon:"success"
				})
				this.getUserInfo();
			}
		}
	}
};
</script>

<style lang="scss" scoped>
page {
	background: #f5f5f5;
}
.meMain {
	width: 100%;
	position: relative;
}
.meMain > image {
	width: 100%;
	height: 220upx;
	display: block;
}
.meMainBox {
	width: 92%;
	margin: 0 auto;
}
.mePosition {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
}
.meHead {
	overflow: hidden;
	position: relative;
}
.meHeadAvatar {
	float: left;
	width: 40%;
	margin-top: 30upx;
}
.meHeadAvatar image {
	width: 150upx;
	height: 150upx;
	display: block;
	border-radius: 50%;
}
.meHeadName {
	float: left;
	width: 60%;
	margin-top: 20upx;
	line-height: 100upx;
	color: #000000;
	font-size: 50upx;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}

.meOverBg {
	background: #ffffff;
	overflow: hidden;
	border-radius: 12upx;
	margin-top: 30upx;
}

.success {
	font-size: 35upx;
	color: #dd524d;
}
.unsuccess {
	font-size: 35upx;
	color: #aaaaaa;
}
.modify {
	margin-top: 50upx;
	margin-bottom: 50upx;
}
</style>
