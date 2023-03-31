<template>
    <view>
        <map id="map" :longitude="longitude" :latitude="latitude" :style="contentStyle" scale="15" :enable-traffic="false" :show-location="true" :enable-poi="true" class="map">
            <cover-image class="location" src="../../static/workbench/location.png" @tap="returnLocationHandle()"></cover-image>
        </map>
        <view class="panel">
            <view class="from" @tap="chooseLocationHandle('from')">
                <text>{{ from.address }}</text>
            </view>
            <view class="dashed-line"></view>
            <view class="to" @tap="chooseLocationHandle('to')">
                <text>{{ to.address }}</text>
            </view>
        </view>
    </view>
</template>

<script>
const chooseLocation = requirePlugin('chooseLocation');
let QQMapWX = require('../../lib/qqmap-wx-jssdk.min.js');

export default {
    data() {
        return {
            from: {
                address: '',
                longitude: 0,
                latitude: 0
            },
            to: {
                address: '输入你的目的地',
                longitude: 0,
                latitude: 0
            },
            longitude: 116.397505,
            latitude: 39.908675,
            contentStyle: '',
            windowHeight: 0,
            map: null,
            flag: null
        };
    },
    methods: {
        returnLocationHandle: function() {
            this.map.moveToLocation();
        },
        chooseLocationHandle: function(flag) {
            let that = this;
            that.flag = flag;
            let key = that.tencent.map.key; //使用在腾讯位置服务申请的key
            let referer = that.tencent.map.referer; //调用插件的app的名称
            let latitude = that.latitude;
            let longitude = that.longitude;
            let data = JSON.stringify({
                latitude: latitude,
                longitude: longitude
            });
            uni.navigateTo({
                url: `plugin://chooseLocation/index?key=${key}&referer=${referer}&location=${data}`
            });
        }
    },
    onShow: function() {
        let that = this;
        that.map = uni.createMapContext('map');
        let qqmapsdk = new QQMapWX({
            key: that.tencent.map.key
        });

        uni.$on('updateLocation', function(location) {
            console.log(location);
            if (that.flag != null) {
                return;
            }
            let latitude = location.latitude;
            let longitude = location.longitude;
            that.latitude = latitude;
            that.longitude = longitude;
            that.from.latitude = latitude;
            that.from.longitude = longitude;

            qqmapsdk.reverseGeocoder({
                location: {
                    latitude: latitude,
                    longitude: longitude
                },
                success: function(resp) {
                    //console.log(resp);
                    that.from.address = resp.result.address;
                },
                fail: function(error) {
                    console.log(error);
                }
            });
        });
        let location = chooseLocation.getLocation();
        if (location != null) {
            let place = location.name;
            let latitude = location.latitude;
            let longitude = location.longitude;
            if (that.flag == 'from') {
                that.from.address = place;
                that.from.latitude = latitude;
                that.from.longitude = longitude;
            } else {
                that.to.address = place;
                that.to.latitude = latitude;
                that.to.longitude = longitude;
                //跳转到创建订单页面
                uni.setStorageSync('from', that.from);
                uni.setStorageSync('to', that.to);
                uni.navigateTo({
                    url: '../create_order/create_order'
                });
            }
        }
    },
    onHide: function() {
        uni.$off('updateLocation');
        //清除地图选点的结果
        chooseLocation.setLocation(null);
    },
    onLoad: function() {
        let that = this;
        let windowHeight = uni.getSystemInfoSync().windowHeight;
        that.windowHeight = windowHeight;
        that.contentStyle = `height:${that.windowHeight}px;`;
		
        //查询乘客当前订单
		that.ajax(that.url.hasCustomerCurrentOrder,"POST",{},function(resp){
			let result=resp.data.result
			let hasCustomerUnAcceptOrder = result.hasCustomerUnAcceptOrder;
			let hasCustomerUnFinishedOrder = result.hasCustomerUnFinishedOrder;
			if(hasCustomerUnAcceptOrder){
				let json=result.unAcceptOrder
				let carType = json.carType;
				let carPlate = json.carPlate;
				let startPlaceLocation = JSON.parse(json.startPlaceLocation);
				let endPlaceLocation = JSON.parse(json.endPlaceLocation);
				let from = {
				    address: json.startPlace,
				    latitude: startPlaceLocation.latitude,
				    longitude: startPlaceLocation.longitude
				};
				let to = {
				    address: json.endPlace,
				    latitude: endPlaceLocation.latitude,
				    longitude: endPlaceLocation.longitude
				};
				uni.setStorageSync("from",from)
				uni.setStorageSync("to",to)
				uni.showModal({
				    title: '提示消息',
				    content: '您有一个订单等待司机接单，现在将跳转到等待接单页面',
				    showCancel: false,
				    success: function(resp) {
				        if (resp.confirm) {
				            uni.navigateTo({
				                url: `../create_order/create_order?showPopup=true&orderId=${json.id}&showCar=true&carType=${carType}&carPlate=${carPlate}`
				            });
				        }
				    }
				});
			}
			else if(hasCustomerUnFinishedOrder){
				uni.showModal({
				    title: '提示消息',
				    content: '您有一个正在执行的定订单，现在将跳转到司乘同显画面',
				    showCancel: false,
				    success: function(resp) {
				        if (resp.confirm) {
				            uni.navigateTo({
				                url: '../move/move?orderId=' + result.unFinishedOrder
				            });
				        }
				    }
				});
			}
		},false)
    },
    onUnload: function() {
        //清除地图选点的结果
        chooseLocation.setLocation(null);
    }
};
</script>

<style lang="less">
@import url('workbench.less');
</style>
