<template>
	<view>
		<view class="cu-list menu-avatar">
			<view class="cu-item" @click="adminInfo">
				<view class="cu-avatar round lg" style="background-image:url(https://beichenhpy.cn/img/user/default.jpg);"></view>
				<view class="content">
					<view class="text-grey">系统通知</view>
					<view v-if="adminUnread != 0" class="cu-tag badge">{{ adminUnread }}</view>
				</view>
			</view>
		</view>

		<view class="cu-list menu-avatar">
			<view class="cu-item" v-for="(items, index) in friendChatList" :key="index" @click="userComment(items)">
				<view class="cu-avatar round lg" :style="{ backgroundImage: 'url(' + items.profilePhoto + ')' }"></view>
				<view class="content">
					<view class="text-grey">{{ items.nickName }}</view>
					<view v-if="items.unRead != 0" class="cu-tag badge">{{ items.unRead }}</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
export default {
	data() {
		return {
			userInfo: {},
			access_token: '',
			friendChatList: [],
			adminUnread: 0
		};
	},
	onLoad() {
		_this = this;
	},
	onShow() {
		this.userInfo = uni.getStorageSync('userInfo');
		this.access_token = uni.getStorageSync('access_token');
		let ok = this.checkToken(this.access_token);
		if (ok) {
			_this.getFriendList();
		}
	},
	methods: {
		checkToken(token) {
			if (token === '') {
				uni.showToast({
					icon: 'none',
					title: '尚未登录，跳转登录页'
				});
				setTimeout(function() {
					uni.navigateTo({
						url: '../login/login?to=1'
					});
				}, 1000);
				return false;
			} else {
				return true;
			}
		},
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
		getFriendList() {
			uni.request({
				url: http.BaseUrl + 'comment/getFriends',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.friendChatList = res.data.data.friendInfos;
						_this.adminUnread = res.data.data.adminUnRead;
						uni.setStorageSync('friendChatList', res.data.data.friendInfos);
					}
				}
			});
		},
		adminInfo() {
			uni.request({
				url: http.BaseUrl + 'comment/clearAdmin',
				method: 'PUT',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.getFriendList();
						uni.navigateTo({
							url: 'adminComment'
						});
					}
				}
			});
		},
		userComment(items) {
			uni.request({
				url: http.BaseUrl + 'comment/clearUnRead/' + items.uid,
				method: 'PUT',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.getFriendList();
						uni.navigateTo({
							url: 'useComment?sender=' + JSON.stringify(items)
						});
					}
				}
			});
		}
	}
};
</script>

<style></style>
