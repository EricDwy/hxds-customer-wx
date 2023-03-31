<template>
    <view>
        <view class="address-container">
            <view class="from">
                <text>{{ from.address }}</text>
            </view>
            <view class="dashed-line"></view>
            <view class="to">
                <text>{{ to.address }}</text>
            </view>
        </view>
        <map
            id="map"
            :longitude="from.longitude"
            :latitude="from.latitude"
            :style="contentStyle"
            scale="12"
            :enable-traffic="false"
            :show-location="true"
            class="map"
            :polyline="polyline"
            :markers="markers"
        ></map>

        <view class="panel">
            <view class="row">
                <view class="info">
                    <view class="label">全程</view>
                    <view class="value">
                        <text>{{ distance }}</text>
                        公里
                    </view>
                    <view class="label">，预计</view>
                    <view class="value">
                        <text>{{ duration }}</text>
                        分钟
                    </view>
                </view>
                <view class="opt" @tap="chooseCarHandle" v-if="!showCar">选择车辆</view>
                <view class="opt" @tap="chooseCarHandle" v-if="showCar">{{ carPlate }}</view>
            </view>
            <button class="btn" @tap="createOrderHandle">确定下单</button>
        </view>
        <u-popup v-model="showPopup" mode="center" width="600" height="350" :mask-close-able="false">
            <view class="popup-title">您的订单正在等待司机接单</view>
            <view class="count-down">
                <u-count-down
                    ref="uCountDown"
                    :timestamp="timestamp"
                    :autoplay="false"
                    separator="zh"
                    :show-hours="false"
                    :show-border="true"
                    bg-color="#DDF0FF"
                    separator-color="#0083F3"
                    border-color="#0D90FF"
                    color="#0D90FF"
                    font-size="32"
                    @end="countEndHandle"
                    @change="countChangeHandle"
                ></u-count-down>
            </view>
            <button class="btn" @tap="cancelHandle">取消订单</button>
        </u-popup>
    </view>
</template>

<script>
let QQMapWX = require('../../lib/qqmap-wx-jssdk.min.js');
let qqmapsdk;
export default {
    data() {
        return {
            from: {
                address: '',
                longitude: 0,
                latitude: 0
            },
            to: {
                address: '',
                longitude: 0,
                latitude: 0
            },
            contentStyle: '',
            windowHeight: 0,
            distance: 0,
            duration: 0,
            polyline: [],
            markers: [],
            infoStatus: true,
            carId: null,
            carPlate: null,
            carType: null,
            showCar: false,
            showPopup: false,
            timestamp: 60,
            orderId: null
        };
    },
    methods: {
        formatPolyline(polyline) {
            let coors = polyline;
            let pl = [];
            //坐标解压（返回的点串坐标，通过前向差分进行压缩）
            const kr = 1000000;
            for (let i = 2; i < coors.length; i++) {
                coors[i] = Number(coors[i - 2]) + Number(coors[i]) / kr;
            }
            //将解压后的坐标放入点串数组pl中
            for (let i = 0; i < coors.length; i += 2) {
                pl.push({
                    longitude: coors[i + 1],
                    latitude: coors[i]
                });
            }
            return pl;
        },
        calculateLine: function(ref) {
            qqmapsdk.direction({
                mode: 'driving',
                from: {
                    latitude: ref.from.latitude,
                    longitude: ref.from.longitude
                },
                to: {
                    latitude: ref.to.latitude,
                    longitude: ref.to.longitude
                },
                success: function(resp) {
                    if (resp.status != 0) {
                        uni.showToast({
                            icon: 'error',
                            title: resp.message
                        });
                        return;
                    }
                    let route = resp.result.routes[0];
                    let distance = route.distance;
                    let duration = route.duration;
                    let polyline = route.polyline;
                    ref.distance = Math.ceil((distance / 1000) * 10) / 10;
                    ref.duration = duration;
                    let points = ref.formatPolyline(polyline);

                    ref.polyline = [
                        {
                            points: points,
                            width: 6,
                            color: '#05B473',
                            arrowLine: true
                        }
                    ];
                    ref.markers = [
                        {
                            id: 1,
                            latitude: ref.from.latitude,
                            longitude: ref.from.longitude,
                            width: 25,
                            height: 35,
                            anchor: {
                                x: 0.5,
                                y: 0.5
                            },
                            iconPath: 'https://mapapi.qq.com/web/lbs/javascriptGL/demo/img/start.png'
                        },
                        {
                            id: 2,
                            latitude: ref.to.latitude,
                            longitude: ref.to.longitude,
                            width: 25,
                            height: 35,
                            anchor: {
                                x: 0.5,
                                y: 0.5
                            },
                            iconPath: 'https://mapapi.qq.com/web/lbs/javascriptGL/demo/img/end.png'
                        }
                    ];
                }
            });
        },
        chooseCarHandle: function() {
            uni.navigateTo({
                url: '../car_list/car_list'
            });
        },
        createOrderHandle: function() {
            let that = this;
            if (that.carType == null || that.carPlate == null) {
                uni.showToast({
                    icon: 'error',
                    title: '没有设置代驾车辆'
                });
                return;
            }
            uni.showLoading({
                title: '下单中请稍后'
            });
            setTimeout(function() {
                uni.hideLoading();
            }, 60000);
            let data = {
                startPlace: that.from.address,
                startPlaceLatitude: that.from.latitude,
                startPlaceLongitude: that.from.longitude,
                endPlace: that.to.address,
                endPlaceLatitude: that.to.latitude,
                endPlaceLongitude: that.to.longitude,
                favourFee: '0.0',
                carPlate: that.carPlate,
                carType: that.carType
            };
            that.ajax(that.url.createNewOrder, 'POST', data, function(resp) {
                uni.hideLoading();
                if (resp.data.result.count > 0) {
                    uni.showToast({
                        icon: 'success',
                        title: '订单创建成功'
                    });
                    setTimeout(function() {
                        that.orderId = resp.data.result.orderId;
                        that.showPopup = true;
                        //此处应该是15*60，但是测试中我们等不了15分钟
                        that.timestamp = 60;
                        that.$refs.uCountDown.start();
                    }, 2000);
                } else {
                    uni.showToast({
                        icon: 'none',
                        title: '没有适合接单的司机'
                    });
                }
            });
        },
        searchOrderStatus: function(ref) {
            let data = {
                orderId: ref.orderId
            };
            ref.ajax(
                ref.url.searchOrderStatus,
                'POST',
                data,
                function(resp) {
                    if (resp.data.result == 2) {
                        ref.showPopup = false;
                        ref.timestamp = null;
                        uni.showToast({
                            icon: 'success',
                            title: '司机已接单'
                        });
                        uni.vibrateShort({});
                        setTimeout(function() {
                            uni.redirectTo({
                                url: '../move/move?orderId=' + ref.orderId
                            });
                        }, 3000);
                    } 
                    else if (resp.data.result == 0) {
                        ref.showPopup = false;
                        ref.timestamp = null;
                        uni.showToast({
                            icon: 'success',
                            title: '订单已经关闭'
                        });
                    }
                },
                false
            );
        },
        deleteUnAcceptOrder: function(ref) {
            ref.showPopup = false;
            ref.timestamp = null;
            let data = {
                orderId: ref.orderId
            };
            ref.ajax(ref.url.deleteUnAcceptOrder, 'POST', data, function(resp) {
                let result = resp.data.result;
                console.log(result);
                if (result == '订单取消成功') {
                    uni.showToast({
                        icon: 'success',
                        title: '订单取消成功'
                    });
                    setTimeout(function() {
                        uni.redirectTo({
                            url: '../workbench/workbench'
                        });
                    }, 3000);
                } else {
                    ref.searchOrderStatus(ref);
                }
            });
        },
        countChangeHandle: function(s) {
            let that = this;
            if (s != 0 && s % 5 == 0) {
                that.searchOrderStatus(that);
            }
        },
        countEndHandle: function() {
            let that = this;
            that.deleteUnAcceptOrder(that);
        },
        cancelHandle: function() {
            let that = this;
            that.deleteUnAcceptOrder(that);
        }
    },
    onLoad: function(options) {
        let that = this;
        //设置地图控件的高度适配屏幕高度
        let windowHeight = uni.getSystemInfoSync().windowHeight;
        that.windowHeight = windowHeight;
        that.contentStyle = `height:${that.windowHeight}px;`;

        that.from = uni.getStorageSync('from');
        that.to = uni.getStorageSync('to');
        qqmapsdk = new QQMapWX({
            key: that.tencent.map.key
        });

        that.map = uni.createMapContext('map');

        if (options.hasOwnProperty('showCar')) {
            that.showCar = options.showCar;
            that.carId = options.carId;
            that.carPlate = options.carPlate;
            that.carType = options.carType;
        }
		
		if(options.hasOwnProperty("showPopup")){
			that.timestamp=60
			that.showPopup=true
			that.orderId=options.orderId
			that.$refs.uCountDown.start();
		}


    },
    onShow: function() {
        let that = this;
        that.calculateLine(that);
    }
};
</script>

<style lang="less">
@import url('create_order.less');
</style>
