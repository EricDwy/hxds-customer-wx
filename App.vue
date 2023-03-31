<script>
export default {
	onLaunch: function() {
		//开启GPS后台刷新
		wx.startLocationUpdate({
			success(resp) {
				console.log('开启定位成功');
			},
			fail(resp) {
				console.log('开启定位失败');
			}
		});
		//GPS定位变化就自动提交给后端
		wx.onLocationChange(function(resp) {
			let latitude = resp.latitude;
			let longitude = resp.longitude;
			let location = { latitude: latitude, longitude: longitude };
			//触发自定义事件
			uni.$emit('updateLocation', location);
		});
	},
	onShow: function() {
		console.log('App Show');
	},
	onHide: function() {
		console.log('App Hide');
	}
};
</script>

<style lang="scss">
@import 'uview-ui/index.scss';
</style>
