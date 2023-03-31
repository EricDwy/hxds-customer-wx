<template>
	<view class="page">
		<view v-if="list.length > 0" v-for="one in list" class="row" @tap="chooseOneHandle(one.id, one.carPlate, one.carType)" @longpress="removeHandle(one.id)">
			<view>
				<u-icon name="info-circle-fill" color="#2979ff" size="35" class="icon"></u-icon>
				<text class="car-type">{{ one.carType }}</text>
			</view>
			<text class="car-plate">{{ one.carPlate }}</text>
		</view>
		<image v-if="list.length == 0" src="../../static/car_list/none.jpg" mode="widthFix" class="none"></image>
		<button class="btn" @tap="addHandle">添加车辆</button>
		<u-toast ref="uToast" />
	</view>
</template>

<script>
export default {
	data() {
		return {
			list: []
		};
	},
	methods: {
		loadDataList: function(ref) {
			ref.list = [];
			ref.ajax(ref.url.searchCustomerCarList, 'POST', {}, function(resp) {
				let result = resp.data.result;
				for (let one of result) {
					ref.list.push({
						id: one.id,
						carType: one.carType,
						carPlate: one.carPlate
					});
				}
			});
		},
		addHandle: function() {
			uni.redirectTo({
				url: '../add_car/add_car'
			});
		},
		removeHandle: function(id) {
			let that = this;
			uni.vibrateShort({});
			uni.showModal({
				title: '提示消息',
				content: '是否删除这条车辆信息？',
				success: function(resp) {
					if (resp.confirm) {
						let data = {
							id: id
						};
						that.ajax(that.url.deleteCustomerCarById, 'POST', data, function(resp) {
							if (resp.data.rows == 0) {
								that.$refs.uToast.show({
									title: '删除失败',
									type: 'error'
								});
							} else {
								that.loadDataList(that);
							}
						});
					}
				}
			});
		},
		chooseOneHandle: function(id, carPlate, carType) {
			uni.navigateTo({
				url: `../create_order/create_order?showCar=true&carId=${id}&carPlate=${carPlate}&carType=${carType}`
			});
		}
	},
	onLoad: function() {
		let that = this;
		that.loadDataList(that);
	}
};
</script>

<style lang="less">
@import url('car_list.less');
</style>
