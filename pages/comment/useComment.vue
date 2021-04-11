<template>
	<view style="margin-bottom: 20%;">
		<cu-custom bgColor="bg-white" :isBack="true" :access_token="access_token" :uid="sender.uid">
			<block slot="backText">返回</block>
			<block slot="content">{{ sender.nickName }}</block>
		</cu-custom>
			<view class="cu-chat" v-if="user !== null">
				<view class="cu-item" v-for="(items, index) in user" :key="index" :class="items.sender === userInfo.uid ? 'self' : ''">
					<view v-if="items.sender == sender.uid" class="cu-avatar radius" :style="senderProfilePhoto"></view>
					<view class="main">
						<view class="content shadow">
							<text>{{ items.content }}</text>
						</view>
					</view>
					<view v-if="items.sender == userInfo.uid" class="cu-avatar radius" :style="MyProfilePhoto"></view>
				</view>
			</view>
		<view class="cu-bar foot input" :style="[{ bottom: InputBottom + 'px' }]">
			<input class="solid-bottom" :adjust-position="false" :focus="false" maxlength="300" cursor-spacing="10" @focus="InputFocus" @blur="InputBlur" v-model="sendMsg" />
			<button class="cu-btn bg-green shadow" @click="send">发送</button>
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
			sender: {},
			userInfo: {},
			user: [],
			access_token: '',
			sendMsg: '',
			reciever: '',
			senderProfilePhoto: '',
			MyProfilePhoto: '',
			InputBottom: 0
		};
	},
	onLoad(e) {
		_this = this;
		this.sender = JSON.parse(e.sender);
		console.log(this.sender)
		this.userInfo = uni.getStorageSync('userInfo');
	},
	onShow() {
		this.senderProfilePhoto = 'background-image:url(' + _this.sender.profilePhoto + ');';
		this.MyProfilePhoto = 'background-image:url(' + _this.userInfo.profilePhoto + ');';
		_this.reciever = this.userInfo.uid;
		this.access_token = uni.getStorageSync('access_token');
		goEasy = new GoEasy({
			host: http.goeasyhost, //应用所在的区域地址: 【hangzhou.goeasy.io |singapore.goeasy.io】
			appkey: http.goeasyKey,
			userId: _this.userInfo.uid, //必须指定，否则无法实现客户端上下线监听功能
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
		let userStorage = uni.getStorageSync(_this.sender.uid + '_' + _this.reciever);
		if (userStorage.length != 0) {
			this.user = JSON.parse(userStorage);
		}
		setTimeout(function() {
			_this.recieve();
			uni.pageScrollTo({
				scrollTop:99999999999999999,
				duration:0
			},100)
		}, 50);
	},
	onPullDownRefresh() {
		_this.reciever = this.userInfo.uid;
		let userStorage = uni.getStorageSync(_this.sender.uid + '_' + _this.reciever);
		if (userStorage.length != 0) {
			this.user = JSON.parse(userStorage);
		}
		this.recieve();
		uni.stopPullDownRefresh();
	},
	methods: {
		recieve() {
			setTimeout(function(){
				goEasy.subscribe({
					channel: _this.userInfo.uid, //替换为您自己的channel
					onMessage: function(message) {
						console.log('recieve');
						let chatMsg = JSON.parse(message.content);
						let sender = chatMsg.sender;
						let reciever = chatMsg.reciever;
						let msg = uni.getStorageSync(sender + '_' + reciever);
						if (msg.length != 0) {
							let chatList = JSON.parse(msg);
							let userMsg = new Object();
							userMsg.sender = sender;
							userMsg.content = chatMsg.content;
							chatList.push(userMsg);
							_this.user = chatList;
							uni.setStorageSync(sender + '_' + reciever, JSON.stringify(chatList));
						} else {
							let list = [];
							let userMsg = new Object();
							userMsg.sender = sender;
							userMsg.content = chatMsg.content;
							list.push(userMsg);
							_this.user = list;
							uni.setStorageSync(sender + '_' + reciever, JSON.stringify(list));
						}
						setTimeout(function(){
							uni.pageScrollTo({
								scrollTop:99999999999999999,
								duration:0
							},10)
						})
						
					}
				});
			},100)
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
		InputFocus(e) {
			this.InputBottom = e.detail.height;
		},
		InputBlur(e) {
			this.InputBottom = 0;
		},
		send() {
			if(_this.sendMsg.length == 0 || _this.sendMsg == null){
				uni.showToast({
					title:"请先写点什么",
					icon:"none"
				})
			}else{
				uni.request({
					url: http.BaseUrl + 'comment/send',
					data: {
						reciever: _this.sender.uid,
						content: _this.sendMsg
					},
					method: 'POST',
					header: {
						Authorization: 'bearer ' + _this.access_token
					},
					success: res => {
						_this.checkResponse(res);
						if (res.data.code === 200) {
							let userStorage = uni.getStorageSync(_this.sender.uid + '_' + _this.reciever);
							if (userStorage.length != 0) {
								let userMsgs = JSON.parse(userStorage);
								let userMsg = new Object();
								userMsg.sender = _this.userInfo.uid;
								userMsg.content = _this.sendMsg;
								userMsgs.push(userMsg);
								_this.user = userMsgs;
								uni.setStorageSync(_this.sender.uid + '_' + _this.reciever, JSON.stringify(userMsgs));
							} else {
								let list = [];
								let userMsg = new Object();
								userMsg.sender = _this.userInfo.uid;
								userMsg.content = _this.sendMsg;
								list.push(userMsg);
								_this.user = list;
								uni.setStorageSync(_this.sender.uid + '_' + _this.reciever, JSON.stringify(list));
							}
						}
						setTimeout(function(){
							uni.pageScrollTo({
								scrollTop:999999999999999999,
								duration:100
							})
						},100)
						_this.sendMsg = '';
					}
				});
			}
		}
	}
};
</script>

<style scoped>
page {
	padding-bottom: 100upx;
}
</style>
