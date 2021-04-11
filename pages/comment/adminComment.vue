<template>
	<view>
		<cu-custom bgColor="bg-white" :isBack="true">
			<block slot="backText">返回</block>
			<block slot="content">系统通知</block>
		</cu-custom>
		<view class="cu-chat" v-if="admin !== null">
			<view class="cu-item" v-for="(items,index) in admin" :key="index">
				<view class="cu-avatar radius" style="background-image:url(https://beichenhpy.cn/img/user/default.jpg);"></view>
				<view class="main">
					<view class="content shadow">
						<text>{{items}}</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	var _this;
	var goEasy;
	import GoEasy from 'goeasy';
	import http from '../../common/http.js';
	export default {
		data() {
			return {
				admin:[],
				userInfo:{}
			}
		},
		onLoad() {
			_this = this;
			this.userInfo = uni.getStorageSync('userInfo');
		},
		onShow() {
			goEasy = new GoEasy({
				host: http.goeasyhost, //应用所在的区域地址: 【hangzhou.goeasy.io |singapore.goeasy.io】
				appkey: http.goeasyKey,
				userId: 'admin'+_this.userInfo.uid, //必须指定，否则无法实现客户端上下线监听功能
				onConnected: function() {
					console.log('连接成功！');
				},
				onDisconnected: function() {
					console.log('连接断开！');
				},
				onConnectFailed: function(error) {
					console.log('连接失败或错误！');
				}
			});
			let adminStorage = uni.getStorageSync('admin')
			if(adminStorage.length != 0){
				this.admin = JSON.parse(adminStorage);
			}
			setTimeout(function(){
				_this.receive()
			},50)
		},
		methods: {
			receive(){
				setTimeout(function(){
					uni.pageScrollTo({
						scrollTop:999999999999999999,
						duration:100
					})
				},50)
				goEasy.subscribe({
					channel: 'admin'+_this.userInfo.uid, //替换为您自己的channel
					onMessage: function(message) {
						let adminMsg = uni.getStorageSync('admin');
						let msg = JSON.parse(message.content)
						if (adminMsg != '') {
							let adminList = JSON.parse(adminMsg);
							adminList.push(msg.content);
							_this.admin = adminList
							uni.setStorageSync('admin', JSON.stringify(adminList));
						} else {
							let adminMsg = [];
							adminMsg.push(msg.content);
							_this.admin = adminMsg
							uni.setStorageSync('admin', JSON.stringify(adminMsg));
						}
						setTimeout(function(){
							uni.pageScrollTo({
								scrollTop:999999999999999999,
								duration:100
							})
						},50)
					}
				});
			}
		}
	}
</script>

<style>

</style>
