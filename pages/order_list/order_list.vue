<template>
	<view>
		<view class="order-list" v-for="one in monthList">
			<view class="month">
				<text>{{ one.month }}</text>
				<image src="../../static/order_list/calendar-icon.png" mode="widthFix"></image>
			</view>
			<view class="order" v-for="order in one.orders" @tap="viewOrderHandle(order.id)">
				<view class="top">
					<view class="date">{{ order.acceptTime }}</view>
					<view class="amount-status">
						<text class="amount" v-if="order.hasOwnProperty('realFee')">￥{{ order.realFee }}</text>
						<text :class="order.style">{{ order.status }}</text>
					</view>
				</view>
				<view class="content">
					<view class="from">
						<text>{{ order.startPlace }}</text>
					</view>
					<view class="to">
						<text>{{ order.endPlace }}</text>
					</view>
					<view class="desc">
						<text>订单编号：{{ order.id }}</text>
					</view>
				</view>
			</view>
		</view>
		<u-top-tips ref="uTips"></u-top-tips>
	</view>
</template>

<script>
let dayjs = require('dayjs');
export default {
	data() {
		return {
			monthList: [],
			page: 1,
			length: 50,
			isLastPage: false
		};
	},
	methods: {
		loadPageData: function(ref) {
			let data = {
				page: ref.page,
				length: ref.length
			};
			let status = {
				'1': '等待接单',
				'2': '已接单',
				'3': '司机已到达',
				'4': '开始代驾',
				'5': '结束代驾',
				'6': '未付款',
				'7': '已付款',
				'8': '订单已结束',
				'9': '顾客撤单',
				'10': '司机撤单',
				'11': '事故关闭',
				'12': '其他'
			};
			ref.ajax(ref.url.searchCustomerOrderByPage, 'POST', data, function(resp) {
				let result = resp.data.result;
				let temp = null;
				let orderList = [];
				let monthList = [];
				if (!result.hasOwnProperty('list') || result.list.length == 0) {
					ref.isLastPage = true;
					return;
				}
				for (let one of result.list) {
					if (temp != null && temp != one.month) {
						monthList.push({ month: temp, orders: orderList });
						orderList = [];
					}
					if (one.status == 6) {
						one.style = 'status red';
					} else if (one.status == 7) {
						one.style = 'status green';
					} else if ([9, 10].includes(one.status)) {
						one.style = 'status orange';
					} else {
						one.style = 'status';
					}
					one.status = status[one.status + ''];
					one.acceptTime = dayjs(one.acceptTime, 'YYYY-MM-DD HH-mm-ss').format('MM/DD HH:mm');
					orderList.push(one);
					temp = one.month;
				}
				monthList.push({ month: temp, orders: orderList });
				if (ref.monthList.length == 0) {
					ref.monthList = monthList;
				} else {
					for (let one of monthList) {
						let flag = false;
						for (let temp of ref.monthList) {
							if (one.month == temp.month) {
								temp.orders = temp.orders.concat(one.orders);
								flag = true;
								break;
							}
						}
						if (!flag) {
							ref.monthList.push(one);
						}
					}
				}
			});
		},
		viewOrderHandle: function(orderId) {
			uni.navigateTo({
				url: '../order/order?orderId=' + orderId
			});
		}
	},
	onLoad: function() {
		let that = this;
		that.loadPageData(that);
	},
	onReachBottom: function() {
		let that = this;
		if (that.isLastPage) {
			return;
		}
		that.page = that.page + 1;
		that.loadPageData(that);
	},

	onShow: function() {},
	onHide: function() {}
};
</script>

<style lang="less">
@import url('order_list.less');
</style>
