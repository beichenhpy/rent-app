<template>
	<view style="margin-bottom: 20%;">
		<view v-if="houseList.length === 0" class="flex justify-center" style="margin-top: 20rpx;">暂无数据</view>
		<view v-for="(items, index) in houseList" :key="index">
			<uni-card :title="items.village + items.address" :is-shadow="true" mode="basic">
				<view class="flex justify-between">
					<text class="text-black weight">核验状况:</text>
					<text v-if="items.isCheck == 1" class="text-green">通过核验</text>
					<text v-else-if="items.isCheck == 2" class="text-red">未通过核验,检查信息</text>
					<text v-else class="text-gray">未核验</text>
				</view>
				<view class="flex justify-between">
					<text class="text-black weight">小区:</text>
					<text>{{ items.address }}</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">楼栋:</text>
					<text class="left">{{ items.building }}栋</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">单元:</text>
					<text class="left">{{ items.unit }}单元</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">朝向:</text>
					<text class="left">{{ items.forward }}朝向</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">房间号:</text>
					<text>{{ items.houseNum }}</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">户型:</text>
					<text class="left">{{ items.layout }}</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">面积::</text>
					<text class="left">{{ items.area }}平</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">出租状态:</text>
					<text v-if="items.isRented === 1" class="text-green left">已出租</text>
					<text v-else class="text-gray">未出租</text>
				</view>
				<view class="flex justify-between">
					<text class="weight">发布状态:</text>
					<text v-if="items.isOnRent === 1" class="text-green left">已发布</text>
					<text v-else class="text-gray">未发布</text>
				</view>
				<view style="margin-top: 20rpx;">
					<view class="flex justify-between" style="margin-bottom: 20rpx;">
						<button class="cu-btn round bg-grey" @click="houseInfo(items)">管理详细信息</button>
						<button class="cu-btn round bg-grey" @click="statistics(items)">查看房屋报表</button>
					</view>
					<view class="flex justify-between" style="margin-bottom: 20rpx;">
						<button class="cu-btn round bg-yellow" @click="modifyHouse(items)">修改房屋信息</button>
						<button class="cu-btn round bg-red" @click="deleteHouse(items)">删除房屋信息</button>
					</view>
					<view class="flex justify-between">
						<button class="cu-btn round bg-yellow" @click="downloadStatistics(items)">导出房屋统计信息</button>
						<button class="cu-btn round bg-yellow" @click="openStatistic(items)">打开房屋统计信息</button>
					</view>
				</view>
			</uni-card>
		</view>
		<uni-pagination :show-icon="true" :total="total" :current="query.page" :pageSize="query.size" @change="change" class="pagination"></uni-pagination>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
import uniPagination from '@dcloudio/uni-ui/lib/uni-pagination/uni-pagination.vue';
export default {
	data() {
		return {
			query: {
				page: 1,
				size: 2
			},
			total: 0,
			houseList: [],
			access_token: '',
			userInfo: {}
		};
	},
	onLoad() {
		_this = this;
	},
	onShow() {
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
		let ok = this.checkToken(this.access_token);
		if (ok) {
			this.getHouse();
		}
	},
	components: {
		uniCard,
		uniPagination
	},
	onBackPress(e) {
		if (e.from === 'backbutton') {
			uni.navigateBack({
				delta: 1
			});
			return true;
		}
	},
	onPullDownRefresh() {
		this.query.page = 1;
		this.getHouse();
		uni.stopPullDownRefresh();
	},
	methods: {
		checkToken(access_token) {
			if (access_token === '') {
				uni.showToast({
					title: '未登录',
					icon: 'none'
				});
				setTimeout(function() {
					uni.navigateTo({
						url: '../login/login'
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
		change(e) {
			this.query.page = e.current;
			this.getHouse();
		},
		getHouse() {
			uni.request({
				url: http.BaseUrl + 'house/findAll',
				data: _this.query,
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: function(res) {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.houseList = res.data.data.list;
						for (let i = 0; i < _this.houseList.length; i++) {
							
						}
						_this.total = res.data.data.total;
					}
				}
			});
		},
		modifyHouse(items) {
			if (items.isRented === 1) {
				uni.showToast({
					title: '已经出租不能修改信息',
					icon: 'none'
				});
			} else if(items.isOnRent === 1){
				uni.showToast({
					title: '已经发布不能修改信息',
					icon: 'none'
				});
			}else{
				uni.navigateTo({
					url: 'modifyHouse?house=' + JSON.stringify(items)
				});
			}
		},
		deleteHouse(items) {
			if (items.isRented === 1) {
				uni.showToast({
					title: '已经出租不能删除',
					icon: 'none'
				});
			} else if(items.isOnRent === 1){
				uni.showToast({
					title: '已经发布不能删除',
					icon: 'none'
				});
			}else {
				uni.showModal({
					title: '确定要删除吗？',
					content: '删除不可逆哦',
					success: function(res) {
						if (res.confirm) {
							uni.showLoading({
								title: '请稍后'
							});
							uni.request({
								url: http.BaseUrl + 'house/delete/' + items.hid,
								method: 'DELETE',
								header: {
									Authorization: 'bearer ' + _this.access_token
								},
								complete: function(res) {
									uni.hideLoading();
									_this.checkResponse(res);
									if (res.data.code === 200) {
										uni.showToast({
											title: res.data.data,
											icon: 'success'
										});
										_this.getHouse();
									}
								}
							});
						}
					}
				});
			}
		},
		houseInfo(items) {
			uni.navigateTo({
				url: 'houseInfo?hid=' + items.hid + '&isRented=' + items.isRented + '&isOnRent=' + items.isOnRent
			});
		},
		statistics(items) {
			uni.request({
				url: http.BaseUrl + 'house/findYears/' + items.hid,
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						uni.navigateTo({
							url: '../statistics/statistics?hid=' + items.hid + '&years=' + JSON.stringify(res.data.data)
						});
					}
				}
			});
		},
		downloadStatistics(items){
			uni.showLoading({
				title: '正在导出'
			});
			uni.request({
				url:http.BaseUrl + 'file/printStatistic/'+items.hid,
				success: (res) => {
					_this.checkResponse(res);
					if(res.data.code === 200){
						uni.downloadFile({
							url: res.data.data,
							success: res => {
								uni.hideLoading()
								uni.saveFile({
									tempFilePath: res.tempFilePath,
									success: resp => {
										uni.setStorageSync('statistics' + items.hid, resp.savedFilePath);
										uni.showToast({
											title:"文件保存在:"+resp.savedFilePath,
											icon:"none"
										})
									}
								});
							}
						});
					}
				}
			})
		},
		openStatistic(items){
			let path = uni.getStorageSync('statistics' + items.hid);
			if(path.length == 0){
				uni.showToast({
					title:"尚未导出",
					icon:"none"
				})
			}else{
				uni.openDocument({
					filePath:path
				})
			}
			
		}
	}
};
</script>

<style scoped>
.pagination {
	position: fixed;
	bottom: 45rpx;
	width: 100%;
}
.weight {
	font-weight: bold;
}
.left {
	margin-left: 20rpx;
}
</style>
