<template>
	<view>
		<view v-if="access_token == '' || userInfo == null || years.length == 0" class="flex justify-center">暂无数据</view>
		<view v-else>
			<view class="flex justify-center">
				<view @click="showPicker" style="color: red; margin-left: 20rpx;">{{ year }}点击选择</view>
			</view>
			<echarts :option="price" style="height: 300px;"></echarts>
			<view class="flex center">
				<button class="bg-blue" @click="changeBar">柱形图</button>
				<button class="bg-blue" @click="changeLine">折线图</button>
			</view>
		</view>
		<lb-picker ref="picker1" v-model="year" mode="selector" :list="years" @confirm="handleConfirm"></lb-picker>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js'
import echarts from '@/components/echarts/echarts.vue';
export default {
	data() {
		return {
			access_token:'',
			userInfo:{},
			years:[],
			year:'',
			price: {
				notMerge: true, // 自定义变量：true代表不合并数据，比如从折线图变为柱形图则需设置为true；false或不写代表合并
				tooltip: {
					trigger: 'axis',
					axisPointer: {
						type: 'cross',
						label: {
							backgroundColor: '#6a7985'
						}
					},
					positionStatus: true, // 自定义变量：是否防止tooltip超出画布
					formatterStatus: true, // 自定义变量：是否格式化tooltip，设置为false时下面三项均不起作用
					formatterUnit: '元', // 自定义变量：数值后面的单位
					formatFloat2: false, // 自定义变量：是否格式化为两位小数
					formatThousands: true // 自定义变量：是否添加千分位
				},
				legend: {
					data: ['租金']
				},
				grid: {
					left: '0%',
					right: '8%',
					bottom: '5%',
					containLabel: true
				},
				xAxis: [
					{
						type: 'category',
						boundaryGap: false,
						data: []
					}
				],
				yAxis: [
					{
						type: 'value'
					}
				],
				series: [
					{
						name: '租金',
						type: 'bar',
						stack: '总收入',
						areaStyle: {},
						barMaxWidth:60,
						data: []
					}
				]
			}
		}
	},
	onLoad() {
		_this = this;
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
		if(this.access_token != '' || this.userInfo != null){
			this.getPriceStatistics()
		}
	},
	components:{
		echarts
	},
	methods: {
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
		handleConfirm(e){
			this.year = e.value
			this.getPriceStatistics()
		},
		showPicker(){
			this.$refs.picker1.show();
		},
		getPriceStatistics(){
			uni.request({
				url: http.BaseUrl + 'house/findYearsByUid/' + _this.userInfo.uid,
				method: 'GET',
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: res => {
					_this.checkResponse(res);
					if (res.data.code === 200) {
						_this.years = res.data.data;
						_this.year = _this.years[0];
						uni.request({
							url:http.BaseUrl + 'house/findPricesByYear',
							data:{
								year: _this.year
							},
							method: 'GET',
							header: {
								Authorization: 'bearer ' + _this.access_token
							},
							success: (res) => {
								_this.checkResponse(res);
								if(res.data.code == 200){
									let result = res.data.data;
									let x = []
									let y = []
									for (let i = 0; i < result.length; i++) {
										x.push(result[i].houseName);
										y.push(result[i].prices);
									}
									_this.price.xAxis[0].data = x;
									_this.price.series[0].data = y;
								}
							}
						})
					}
				}
			});
		
		},
		changeBar(){
			this.price.series[0].type = 'bar'
		},
		changeLine(){
			this.price.series[0].type = 'line'
		}
	}
}
</script>

<style></style>
