<template>
	<view class="page">
		<u-cell-group :border="false">
			<u-cell-item icon="info-circle-fill" title="车型信息" :value="carType" @click="carTypeHandle"></u-cell-item>
			<u-cell-item icon="info-circle-fill" title="车牌号码" :value="carPlate" @click="carPlateHandle"></u-cell-item>
		</u-cell-group>
		<button class="btn" @tap="saveHandle">确定保存</button>
		<view class="desc">
			保存即代表您同意华夏代驾的
			<text>《隐私政策》</text>
		</view>
		<u-toast ref="uToast" />
	</view>
</template>

<script>
export default {
	data() {
		return {
			carType: '',
			carPlate: ''
		};
	},
	methods: {
		carTypeHandle:function(){
			let that=this
			uni.showModal({
				title: '输入车型',
				editable: true,
				placeholderText: '例如丰田卡罗拉',
				success: function(resp) {
					if(resp.confirm){
						let carType=resp.content
						if(that.checkValidCarType(carType, '车型')){
							 that.carType = carType;
						}
					}
				}
			})
		},
		carPlateHandle:function(){
			let that = this;
			uni.showModal({
			    title: '输入车牌',
			    editable: true,
			    placeholderText: '你的车牌号',
			    success: function(resp) {
			        if (resp.confirm) {
			            let carPlate = resp.content;
			            if (that.checkValidCarPlate(carPlate, '车牌')) {
			                that.carPlate = carPlate;
			            }
			        }
			    }
			});
		},
		saveHandle:function(){
			let that=this
			if(that.checkValidCarType(that.carType, '车型') && that.checkValidCarPlate(that.carPlate, '车牌')){
				let data={
					carType:that.carType,
					carPlate:that.carPlate
				}
				that.ajax(that.url.insertCustomerCar,"POST",data,function(resp){
					that.$refs.uToast.show({
					    title: '添加成功',
					    type: 'success',
					    callback: function() {
					        uni.redirectTo({
					            url: '../car_list/car_list'
					        });
					    }
					});
				})
			}
		}
	}
};
</script>

<style lang="less">
@import url('add_car.less');
</style>
