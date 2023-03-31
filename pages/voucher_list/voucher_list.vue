<template>
	<view>
		<view class="tabs">
			<u-tabs-swiper
				ref="uTabs"
				:list="list"
				:current="current"
				@change="tabsChange"
				:is-scroll="false"
				:offset="[10, 50]"
				swiperWidth="750"
			/>
		</view>

		<scroll-view scroll-y :style="swipperStyle" @scrolltolower="onreachBottom" v-if="current == 0">
			<view class="voucher" v-for="one in voucherList">
				<view class="left green">
					<view class="row">
						<text class="amount">{{ one.discount }}</text>
					</view>
					<view class="row">
						<text class="desc" v-if="one.withAmount > 0">满{{ one.withAmount }}元可用</text>
						<text class="desc" v-if="one.withAmount == 0">无最低消费</text>
					</view>
				</view>
				<view class="right">
					<view class="row">
						<text class="desc">{{ one.remark }}</text>
					</view>
					<view class="row">
						<text class="desc" v-if="one.timeType == 1">领券后{{ one.days }}天内有效</text>
						<text class="desc" v-if="one.timeType == 2">{{ one.startTime }} ~ {{ one.endTime }}</text>
						<text class="desc" v-if="one.timeType == null">{{ one.createTime }} ~ 2099-01-01</text>
					</view>
				</view>
			</view>
		</scroll-view>

		<scroll-view scroll-y :style="swipperStyle" @scrolltolower="onreachBottom" v-if="current == 1">
			<view class="voucher" v-for="one in voucherList">
				<view class="left orange">
					<view class="row">
						<text class="amount">{{ one.discount }}</text>
					</view>
					<view class="row">
						<text class="desc" v-if="one.withAmount > 0">满{{ one.withAmount }}元可用</text>
						<text class="desc" v-if="one.withAmount == 0">无最低消费</text>
					</view>
				</view>
				<view class="right">
					<view class="row">
						<text class="desc">{{ one.remark }}</text>
					</view>
					<view class="row">
						<text class="desc" v-if="one.timeType == 1">领券后{{ one.days }}天内有效</text>
						<text class="desc" v-if="one.timeType == 2">{{ one.startTime }} ~ {{ one.endTime }}</text>
						<text class="desc" v-if="one.timeType == null">{{ one.createTime }} ~ 2099-01-01</text>
					</view>
				</view>
			</view>
		</scroll-view>

		<scroll-view scroll-y :style="swipperStyle" @scrolltolower="onreachBottom" v-if="current == 2">
			<view class="voucher" v-for="one in voucherList" @tap="takeVoucher(one.id, one.uuid)">
				<view class="left red">
					<view class="row">
						<text class="amount">{{ one.discount }}</text>
					</view>
					<view class="row">
						<text class="desc" v-if="one.withAmount > 0">满{{ one.withAmount }}元可用</text>
						<text class="desc" v-if="one.withAmount == 0">无最低消费</text>
					</view>
				</view>
				<view class="right">
					<view class="row">
						<text class="desc">{{ one.remark }}</text>
					</view>
					<view class="row">
						<text class="desc" v-if="one.timeType == 1">领券后{{ one.days }}天内有效</text>
						<text class="desc" v-if="one.timeType == 2">{{ one.startTime }} ~ {{ one.endTime }}</text>
						<text class="desc" v-if="one.timeType == null">{{ one.createTime }} ~ 2099-01-01</text>
					</view>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			list: [
				{
					name: '未使用',
					count: 0
				},
				{
					name: '已使用',
					count: 0
				},
				{
					name: '待领取',
					count: 0
				}
			],
			current: 0, // tabs组件的current值，表示当前活动的tab选项
			swipperStyle: '',
			page: 1,
			length: 10,
			voucherList: [],
			isLastPage: false
		};
	},
	methods: {
		loadPageData: function(ref) {
			let data = {
				page: ref.page,
				length: ref.length
			};
			if (ref.current == 0) {
				ref.ajax(ref.url.searchUnUseVoucherByPage, 'POST', data, function(resp) {
					let result = resp.data.result;
					ref.list[0].count = result.totalCount;
					if (!result.hasOwnProperty('list') || result.list.length == 0) {
						ref.isLastPage = true;
						uni.showToast({
							icon: 'none',
							title: '已经到底了'
						});
						return;
					}
					let list = result.list;
					for (let one of list) {
						ref.voucherList.push(one);
					}
				});
			} else if (ref.current == 1) {
				ref.ajax(ref.url.searchUsedVoucherByPage, 'POST', data, function(resp) {
					let result = resp.data.result;
					if (!result.hasOwnProperty('list') || result.list.length == 0) {
						ref.isLastPage = true;
						uni.showToast({
							icon: 'none',
							title: '已经到底了'
						});
						return;
					}
					let list = result.list;
					for (let one of list) {
						ref.voucherList.push(one);
					}
				});
			} else if (ref.current == 2) {
				ref.ajax(ref.url.searchUnTakeVoucherByPage, 'POST', data, function(resp) {
					let result = resp.data.result;
					if (!result.hasOwnProperty('list') || result.list.length == 0) {
						ref.isLastPage = true;
						uni.showToast({
							icon: 'none',
							title: '已经到底了'
						});
						return;
					}
					let list = result.list;
					for (let one of list) {
						ref.voucherList.push(one);
					}
				});
			}
		},
		tabsChange: function(index) {
			this.voucherList = [];
			this.$refs.uTabs.setFinishCurrent(index);
			this.current = index;
			this.isLastPage = false;
			this.page = 1;
			this.loadPageData(this);
		},
		onreachBottom: function() {
			if (this.isLastPage) {
				return;
			}
			this.page = this.page + 1;
			this.loadPageData(this);
		},
		takeVoucher: function(id, uuid) {
			let that = this;
			uni.showModal({
				title: '提示信息',
				content: '领取这张代金券？',
				success: function(resp) {
					if (resp.confirm) {
						let data = {
							id: id,
							uuid: uuid
						};
						that.ajax(that.url.takeVoucher, 'POST', data, function(resp) {
							let result = resp.data.result;
							if (result) {
								uni.showToast({
									icon: 'success',
									title: '领取成功'
								});
								that.list[0].count++;
								setTimeout(function() {
									that.page = 1;
									that.voucherList = [];
									that.isLastPage = false;
									that.loadPageData(that);
								}, 1500);
							} else {
								uni.showToast({
									icon: 'error',
									title: '领取失败'
								});
							}
						});
					}
				}
			});
		}
	},
	onLoad: function() {
		let that = this;
		let windowHeight = uni.getSystemInfoSync().windowHeight;
		that.windowHeight = windowHeight;
		that.swipperStyle = `height:${that.windowHeight - 70}px;`;
		that.loadPageData(that);
	}
};
</script>

<style lang="less">
@import url('voucher_list.less');
</style>
