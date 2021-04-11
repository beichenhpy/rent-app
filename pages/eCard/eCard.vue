<template>
	<view class='contents'>

		<canvas class='firstCanvas' canvas-id="firstCanvas" @touchmove='move' @touchstart='start($event)' @touchend='end'
		 @touchcancel='cancel' @longtap='tap' disable-scroll='true' @error='error'>
		</canvas>

		<view class="flex justify-between">
			<button @click='clearClick'>清除</button>
			<button @click='upload'>上传图片</button>
		</view>

	</view>
</template>

<script>
	var _this;
	var content = null;
	var touchs = [];
	var canvasw = 0;
	var canvash = 0;
	//获取系统信息
	uni.getSystemInfo({
		success: function(res) {
			canvasw = res.windowWidth;
			canvash = canvasw * 9 / 16;
		},
	})
	import http from '../../common/http.js';
	export default {
		data() {
			return {
				access_token: '',
				userInfo: {}
			}
		},
		onLoad() {
			//获得Canvas的上下文
			content = uni.createCanvasContext('firstCanvas')
			//设置线的颜色
			content.setStrokeStyle("#000")
			//设置线的宽度
			content.setLineWidth(5)
			//设置线两端端点样式更加圆润
			content.setLineCap('round')
			//设置两条线连接处更加圆润
			content.setLineJoin('round')
			_this = this;
			this.access_token = uni.getStorageSync('access_token');
			this.userInfo = uni.getStorageSync('userInfo');
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
				if (res.data.code === 400 || res.data.status === 500) {
					uni.showToast({
						icon: 'none',
						title: '服务器开小差了'
					});
				}
			},
			// 画布的触摸移动开始手势响应
			start: function(e) {
				//获取触摸开始的 x,y
				let point = {
					x: e.touches[0].x,
					y: e.touches[0].y,
				}
				// console.log(touchs)
				touchs.push(point);

			},
			// 画布的触摸移动手势响应
			move: function(e) {
				let point = {
					x: e.touches[0].x,
					y: e.touches[0].y
				}
				touchs.push(point)
				if (touchs.length >= 2) {
					this.draw(touchs)
				}
			},

			// 画布的触摸移动结束手势响应
			end: function(e) {
				//清空轨迹数组
				for (let i = 0; i < touchs.length; i++) {
					touchs.pop()
				}

			},

			// 画布的触摸取消响应
			cancel: function(e) {
				console.log("触摸取消" + e)
			},

			// 画布的长按手势响应
			tap: function(e) {
				console.log("长按手势" + e)
			},

			error: function(e) {
				console.log("画布触摸错误" + e)
			},

			//绘制
			draw: function(touchs) {
				let point1 = touchs[0]
				let point2 = touchs[1]
				touchs.shift()
				content.moveTo(point1.x, point1.y)
				content.lineTo(point2.x, point2.y)
				content.stroke()
				content.draw(true)
			},
			//清除操作
			clearClick: function() {
				//清除画布
				content.clearRect(0, 0, canvasw, canvash)
				content.draw(true)
			},
			upload(){
				uni.canvasToTempFilePath({
					canvasId: 'firstCanvas',
					success: (res) =>{
						//打印图片路径
						let savedFilePath=res.tempFilePath;
						console.log(savedFilePath)
						uni.uploadFile({
							url:http.BaseUrl + 'user/updateECard',
							filePath:savedFilePath,
							name:"file",
							header: {
								Authorization: 'bearer ' + _this.access_token
							},
							success:(res)=>{
								let response = JSON.parse(res.data);
								_this.checkResponse(response);
								if(response.code === 200){
									uni.showToast({
										title:"上传成功",
										icon:"success"
									})
									setTimeout(function(){
										uni.navigateBack({
											delta:1
										})
									},500)
								}
							}
						})
					}
				});
				
			}





		}



	}
</script>

<style scoped>
	content {
		width: 100%;
		height: 500px;
	}

	.firstCanvas {
		background-color: #ddd;
		width: 100%;
		height: 200px;
	}

	.contents image {
		width: 100%;
		height: 200px;
		background-color: orange;
	}

	#signatureImg {
		background-color: #ddd;
	}
</style>
