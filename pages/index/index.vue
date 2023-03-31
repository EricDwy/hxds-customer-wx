<template>
	<view class="content">
		<button @tap="click">付款</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			}
		},
		onLoad() {
			
		},
		methods: {
			click:function(){
				uni.requestPayment({
					timeStamp: "1642994349292",
					nonceStr: "LnXEdknTiDl2sB03DoD4lXLhNvYFkoO4",
					package: "prepay_id=wx24111908878403f9d3ce04c7bd00ec0000",
					signType: 'MD5',
					paySign: "AEF64FABE85AD3BE89FEF05DB9573330",
					success: function() {
						uni.showToast({
							title: '支付成功'
						});
					
					},
					fail: function() {
						uni.showToast({
							icon: 'none',
							title: '支付失败'
						});
					}
				});
			}
		}
	}
</script>

<style>

</style>
