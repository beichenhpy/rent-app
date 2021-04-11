<template>
	<view>
		<view v-if="years.length == 0" style="margin-top: 20rpx;" class="flex justify-center">暂无数据</view>
		<view v-else>
			<view class="flex justify-center">
				<view>月报表</view>
				<view @click="showPicker" style="color: red; margin-left: 20rpx;">{{ year }}点击选择</view>
			</view>
			<echarts :option="count" style="height: 300px;"></echarts>
			<echarts :option="price" style="height: 300px;"></echarts>
			<lb-picker ref="picker1" v-model="year" mode="selector" :list="years" @confirm="handleConfirm"></lb-picker>
			<view style="margin-top: 20rpx;">
				<view class="flex justify-center">
					<button class="bg-blue" @click="showYearStatistics">查看年度报表</button>
				</view>
				<view class="flex center">
					<button class="bg-blue" @click="changeBar">柱状图</button>
					<button class="bg-blue" @click="changeLine">折线图</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
var _this;
import http from '../../common/http.js';
import echarts from '@/components/echarts/echarts.vue';
export default {
	data() {
		return {
			hid: '',
			years:[],
			year:'',
			dates:[],
			electCounts:[],
			waterCounts:[],
			electPrices:[],
			waterPrices:[],
			access_token:'',
			userInfo:{},
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
					formatFloat2: true, // 自定义变量：是否格式化为两位小数
					formatThousands: true // 自定义变量：是否添加千分位
				},
				legend: {
					data: ['电费','水费']
				},
				grid: {
					left: '5%',
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
						name: '电费',
						type: 'line',
						stack: '总量',
						areaStyle: {},
						data: []
					},
					{
						name: '水费',
						type: 'line',
						stack: '总量',
						areaStyle: {},
						barMaxWidth:60,
						data: []
					}
				]
			},
			count: {
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
					formatterUnit: '度/吨', // 自定义变量：数值后面的单位
					formatFloat2: false, // 自定义变量：是否格式化为两位小数
					formatThousands: true // 自定义变量：是否添加千分位
				},
				legend: {
					data: ['用电量', '用水量']
				},
				grid: {
					left: '5%',
					right: '8%',
					bottom: '5%',
					containLabel: true
				},
				xAxis: [
					{
						 axisLabel: {
						                interval:0//代表显示所有x轴标签显示
						            },
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
						name: '用电量',
						type: 'line',
						stack: '总量',
						areaStyle: {},
						data: []
					},
					{
						name: '用水量',
						type: 'line',
						stack: '总量',
						areaStyle: {},
						barMaxWidth:60,
						data: []
					}
				]
			}
		};
	},
	components: {
		echarts
	},
	onLoad(e) {
		_this = this;
		this.hid = e.hid;
		this.years = JSON.parse(e.years);
		this.access_token = uni.getStorageSync('access_token');
		this.userInfo = uni.getStorageSync('userInfo');
	},
	onShow() {
		this.year = this.years[0]
		this.getElectAndWater();
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
		getElectAndWater() {
			uni.request({
				url:http.BaseUrl + 'house/findElectWaterByYear/'+_this.hid,
				method:"GET",
				data:{
					year:_this.year
				},
				header: {
					Authorization: 'bearer ' + _this.access_token
				},
				success: (res) => {
					_this.checkResponse(res);
					if(res.data.code === 200){
						let result = res.data.data;
						_this.count.xAxis[0].data = result.date;
						_this.price.xAxis[0].data = result.date;
						_this.count.series[0].data = result.electCount;
						_this.count.series[1].data = result.waterCount;
						_this.price.series[0].data = result.electPrice;
						_this.price.series[1].data = result.waterPrice;
					}
				}
			})
		},
		showPicker(){
			this.$refs.picker1.show();
		},
		handleConfirm(e){
			this.year = e.value
			this.getElectAndWater()
		},
		showYearStatistics(){
			uni.navigateTo({
				url:"yearStatistics?years="+JSON.stringify(_this.years)+"&hid="+_this.hid
			})
		},
		changeBar(){
			this.count.series[0].type = 'bar'
			this.count.series[1].type = 'bar'
			this.price.series[0].type = 'bar'
			this.price.series[1].type = 'bar'
		},
		changeLine(){
			this.count.series[0].type = 'line'
			this.count.series[1].type = 'line'
			this.price.series[0].type = 'line'
			this.price.series[1].type = 'line'
		}
	}
};
</script>

<style></style>
