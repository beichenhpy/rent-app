<template>
	<view style="background-color: #000000;">
		<view><video :style="videos" :src="video" :autoplay="true"></video></view>
	</view>
</template>

<script>
	var _this;
export default {
	data() {
		return {
			video: '',
			videos:{
				width:'100%',
				height:'500rpx'
			}
		};
	},
	onLoad(url) {
		_this = this;
		this.video = url.video;
	},
	methods: {
		
	}
};
</script>

<style scoped>
</style>
