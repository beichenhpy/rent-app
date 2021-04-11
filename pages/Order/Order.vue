<template>
	<view>
		<scroll-view scroll-x class="bg-red nav text-center">
			<view class="cu-item" :class="0==TabCur?'text-white cur':''" @tap="myOrder" data-id="0">
				我的订单
			</view>
			<view class="cu-item" :class="1==TabCur?'text-white cur':''" @tap="myRent" data-id="1">
				我的出租
			</view>
		</scroll-view>
		<!-- renter -->
		<view class="cu-list menu" v-for="(items, index) in orderList" :key="index" style="margin-top: 20rpx;" >
			<view class="cu-item arrow" @tap="orderInfo(items.oid,items.hid)">
				<view class="content">
					<view class="text-grey flex justify-between">
						<text>开始时间:{{items.startTime}}</text>
						<text>结束时间:{{items.endTime}}</text>
					</view>
					<view class="text-grey flex justify-start">
						<text>订单状态:</text>
						<text :class="items.isFinish === 1 ? 'order-finish' :'order-unfinish'">{{items.isFinish === 1 ? '已完成' :'未完成'}}</text>
					</view>
				</view>
			</view>
		</view>
			<uni-pagination :show-icon="true" :total="total" :current="query.page" :pageSize="query.size" @change="change" class="pagination"></uni-pagination>
	</view>
</template>

<script>
	var _this;
	import http from '../../common/http.js'
	import uniPagination from '@dcloudio/uni-ui/lib/uni-pagination/uni-pagination.vue';
	export default {
		data() {
			return {
				TabCur: 0,
				query:{
					type:'renter',
					page:1,
					size:5
				},
				orderList:[],
				total:0
			};
		},
		components:{
			uniPagination
		},
		onShow() {
			this.getOrder()
		},
		onLoad() {
			_this = this;
		},
		methods: {
			checkToken(token){
				if(token === ''){
					uni.showToast({
						icon:"none",
						title:"尚未登录，跳转登录页"
					})
					setTimeout(function(){
						uni.navigateTo({
							url:"../login/login?to=1"
						})
					},1000);
					return false;
				}else{
					return true;
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
			getOrder(){
				let access_token = uni.getStorageSync('access_token');
				
				let ok = this.checkToken(access_token);
				if(ok){
					uni.request({
						url:http.BaseUrl + 'order/findAll',
						method:"GET",
						data: _this.query,
						header: {
							Authorization: 'bearer ' + access_token
						},
						success:function(res){
							_this.checkResponse(res);
							if(res.data.code === 200){
								_this.orderList = res.data.data.list;
								_this.total = res.data.data.total;
							}
						}
					})
				}
			},
			change(e){
				this.query.page = e.current;
				this.getOrder()
			},
			myOrder(e) {
				this.TabCur = e.currentTarget.dataset.id;
				this.query.type = 'renter'
				this.getOrder()
			},
			myRent(e){
				this.TabCur = e.currentTarget.dataset.id;
				this.query.type = 'owner'
				this.getOrder()
			},
			orderInfo(oid,hid){
				//根据query.type跳转
				if(this.query.type === 'renter'){
					uni.navigateTo({
						url:"orderInfoRenter?oid="+oid+"&hid="+hid
					})
				}else{
					uni.navigateTo({
						url:"orderInfoOwner?oid="+oid+"&hid="+hid
					})
				}
			}
		}
	}
</script>

<style scoped>
.pagination{
	position: fixed;
	bottom: 0rpx;
	width: 100%;
}
.order-finish{
	margin-left: 20rpx;
	 color: green; 
	 font-weight: bold;
}
.order-unfinish{
	margin-left: 20rpx;
	 color: red; 
	 font-weight: bold;
}
</style>
