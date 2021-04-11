<template>
	<view style="margin-bottom: 20%;">
		<region-picker @regions="getRegion"></region-picker>
		<view v-for="(items, index) in rent" :key="index">
			<uni-card :title="items.title" :is-shadow="true" :extra="items.createTime" note="true" mode="basic" @click="getRentInfo(items.hid, items.uid)">
				<view>
					<text style="font-size: 30upx; color: #BD1E2C;">{{ items.introduction }}</text>
				</view>
				<view><image :src="items.image" mode="aspectFit"></image></view>
				<view class="contain">{{ items.detailPosition }}</view>
				<template v-slot:footer>
					<view class="footer">
						<view class="footer_item">租金:{{ items.price }}元/月</view>
						<view class="footer_item">{{ items.leastTime }}个月起租</view>
					</view>
					<view class="footer_item">押金:{{ items.deposit + items.keyDeposit }}元(包括钥匙押金{{ items.keyDeposit }}元)</view>
				</template>
			</uni-card>
		</view>
		<uni-pagination :show-icon="true" :total="total" :current="query.page" :pageSize="query.size" @change="change" class="pagination"></uni-pagination>
		<uni-fab :pattern="pattern" :content="content" @trigger="addRent" horizontal="right"></uni-fab>
	</view>
</template>

<script>
var _this;
import regionPicker from '../../components/RegionPicker/RegionPicker.vue';
import http from '../../common/http.js';
import uniPagination from '@dcloudio/uni-ui/lib/uni-pagination/uni-pagination.vue';
import uniFab from '@dcloudio/uni-ui/lib/uni-fab/uni-fab.vue';
import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue';
export default {
	data() {
		return {
			query: {
				page: 1,
				size: 5,
				province: '广西壮族自治区',
				city: '南宁市'
			},
			rent: [],
			total: 0,
			pattern: {
				buttonColor: 'red'
			},
			content: [
				{
					iconPath: '/static/rent.png',
					text: '发布出租',
					active: false
				},
				{
					iconPath: '/static/tabBar/index.png',
					text: '添加房屋',
					active: false
				}
			],
			ok: false
		};
	},
	onLoad() {
		_this = this;
		this.getRent();
		if (this.ok) {
			setTimeout(function() {
				uni.showToast({
					title: '共找到' + _this.total + '条出租信息',
					icon: 'none'
				});
			}, 1000);
		}
	},
	onShow() {
		this.getRent();
	},
	onPullDownRefresh() {
		this.query = {
			page: 1,
			size: 5,
			province: '广西壮族自治区',
			city: '南宁市'
		};
		this.getRent();
		if (this.ok) {
			setTimeout(function(){
				uni.showToast({
					title: '共找到' + _this.total + '条出租信息',
					icon: 'none'
				});
			},50)
		}
		uni.stopPullDownRefresh();
	},
	components: {
		uniPagination,
		regionPicker,
		uniCard,
		uniFab
	},
	methods: {
		getRegion(item) {
			this.query.province = item.value[0];
			this.query.city = item.value[1];
			this.getRent();
		},
		change(e) {
			this.query.page = e.current;
			this.getRent();
		},
		getRent() {
			uni.request({
				url: http.BaseUrl + 'rent/findAll',
				data: _this.query,
				method: 'GET',
				success: function(response) {
					if (response.data.code === 200) {
						_this.rent = response.data.data.list;
						_this.total = response.data.data.total;
						_this.ok = true;
					} else {
						uni.showToast({
							title: '服务器开小差了，稍后再试',
							icon: 'none'
						});
					}
				},
				fail: function() {
					uni.showToast({
						title: '服务器开小差了，稍后再试',
						icon: 'none'
					});
				}
			});
		},
		getRentInfo(hid, uid) {
			uni.navigateTo({
				url: '../rentInfo/rentInfo?hid=' + hid + '&uid=' + uid
			});
		},
		addRent(e) {
			if (uni.getStorageSync('access_token') === '') {
				//判断是否登录
				uni.showToast({
					icon: 'none',
					title: '你还没登录呦'
				});
				setTimeout(function() {
					uni.navigateTo({
						url: '../login/login?to=1'
					});
				}, 1000);
			} else {
				if (e.index === 0) {
					uni.redirectTo({
						url: '../addRent/addRent'
					});
				}else{
					uni.redirectTo({
						url:"../house/addHouse"
					})
				}
			}
		}
	}
};
</script>

<style scoped lang="scss">
.chooseRegion {
	color: #aaa899;
	font-size: 30upx;
	background-color: #ffffff;
	line-height: 97upx;
}
.chooseRegion::after {
	border: none;
}
.contain {
	font-size: 30upx;
	color: #aa0000;
}

.footer {
	display: flex;
	justify-content: space-between;
	flex-direction: row;
}

.footer_item {
	color: #aaa899;
	font-size: 30upx;
	align-items: center;
	padding: 10upx 0;
}
.pagination{
	position: fixed;
	bottom: 0rpx;
	width: 100%;
}
</style>
