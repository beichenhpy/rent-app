<template>
	<view class="forget">
		
		<view class="content">
			<!-- 主体 -->
			<view class="main">
				<view class="tips">若你忘记了密码，可在此重置新密码。</view>
				<wInput
					v-model="userData.username"
					type="text"
					maxlength="11"
					placeholder="请输入手机号码"
				></wInput>
				<wInput
					v-model="userData.password"
					type="password"
					maxlength="11"
					placeholder="请输入新密码"
					isShowPass
				></wInput>
				
				<wInput
					v-model="userData.verCode"
					type="number"
					maxlength="6"
					placeholder="验证码"
					
					isShowCode
					codeText="获取重置码"
					setTime="60"
					ref="runCode"
					@setCode="getVerCode()"
				></wInput>
			</view>
			
			<wButton 
				text="重置密码"
				:rotate="isRotate" 
				@click.native="startRePass()"
			></wButton>

		</view>
	</view>
</template>

<script>
	var _this;
	import wInput from '../../components/watch-login/watch-input.vue' //input
	import wButton from '../../components/watch-login/watch-button.vue' //button
	import http from '../../common/http.js'
	export default {
		data() {
			return {
				userData: {
					username: '',
					password: '',
					verCode: ''
				},
				isRotate: false, //是否加载旋转
			}
		},
		components:{
			wInput,
			wButton
		},
		mounted() {
			_this= this;
		},
		methods: {
			getVerCode(){
				//获取验证码
				if (!(/^1[3456789]\d{9}$/.test(_this.userData.username))) {
					uni.showToast({
						icon: 'none',
						position: 'bottom',
						title: '手机号不正确'
					});
					return false;
				}
				uni.request({
					url: http.BaseUrl + 'user/sendSms/' + _this.userData.username,
					method: 'POST',
					success: function(res) {
						if (res.data.code === 200) {
							_this.$refs.runCode.$emit('runCode'); //触发倒计时（一般用于请求成功验证码后调用）
							uni.showToast({
								icon: 'loading',
								position: 'bottom',
								title: '短信已经发送'
							});
							//1分钟终止倒计时
							setTimeout(function() {
								_this.$refs.runCode.$emit('runCode', 0);
							}, 60000);
						} else {
							uni.showToast({
								icon: 'none',
								position: 'bottom',
								title: '短信发送失败'
							});
						}
					}
				});
			},
			startRePass() {
				//重置密码
				if(this.isRotate){
					//判断是否加载中，避免重复点击请求
					return false;
				}
				if (!(/^1[3456789]\d{9}$/.test(_this.userData.username))) {
				    uni.showToast({
				        icon: 'none',
						position: 'bottom',
				        title: '手机号不正确'
				    });
				    return false;
				}
			    if (this.userData.password.length < 6) {
			        uni.showToast({
			            icon: 'none',
						position: 'bottom',
			            title: '密码不正确'
			        });
			        return false;
			    }
				if (this.userData.verCode.length != 6) {
				    uni.showToast({
				        icon: 'none',
						position: 'bottom',
				        title: '验证码不正确'
				    });
				    return false;
				}
				_this.isRotate=true
				uni.request({
					url:http.BaseUrl+'user/forgetPassword',
					method:"PUT",
					data:_this.userData,
					success:function(res){
						if(res.data.code === 200){
							uni.showToast({
								icon:"success",
								title:res.data.data
							})
							uni.clearStorageSync();
							uni.redirectTo({
								url:"login"
							})
							_this.isRotate=false;
						}else{
							uni.showToast({
								icon:"none",
								title:"该用户不存在"
							})
							_this.isRotate = false;
						}
					},fail:function(res){
						uni.showToast({
							icon:"none",
							title:'服务器开小差了'
						})
						_this.isRotate = false;
					}
				})
				
				
			}
		}
	}
</script>

<style>
	@import url("../../components/watch-login/css/icon.css");
	@import url("./css/main.css");
</style>

