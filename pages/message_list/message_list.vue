<template>
	<view>
		<view class="message-container" v-for="one in list" v-if="list.length > 0" @tap="viewMessageHandle(one.id, one.readFlag, one.refId)">
			<view class="top">
				<view class="title">
					<image src="../../static/message_list/email-icon-1.png" mode="widthFix" v-if="!one.readFlag"></image>
					<image src="../../static/message_list/email-icon-2.png" mode="widthFix" v-if="one.readFlag"></image>
					<text>{{ one.senderName }}</text>
				</view>
				<view class="date">{{ one.sendTime }}</view>
			</view>
			<view class="content">{{ one.msg }}</view>
			<view class="bottom">
				<text>查看详情</text>
				<u-icon name="arrow-right" color="#aaa" size="28"></u-icon>
			</view>
		</view>
		<view class="empty" v-if="list.length == 0"><u-empty text="消息列表为空" mode="list"></u-empty></view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			page: 1,
			length: 50,
			list: [],
			isLastPage: false
		};
	},
	methods: {
		loadPageData: function(ref) {
			let data = {
				page: ref.page,
				length: ref.length
			};
			ref.ajax(ref.url.searchMessageByPage, 'POST', data, function(resp) {
				// console.log(resp);
				let result = resp.data.result;
				// console.log(result);
				if (result == null || result.length == 0) {
					ref.isLastPage = true;
					return;
				} else {
					if (ref.page == 1) {
						ref.list = [];
					}
					ref.list = ref.list.concat(result);
				}
				ref.list = result;
			});
		},
		viewMessageHandle: function(id, readFlag, refId) {
			uni.navigateTo({
				url: `../message/message?id=${id}&&readFlag=${readFlag}&&refId=${refId}`
			});
		}
	},
	// onLoad: function() {
	// 	let that = this;
	// 	that.loadPageData(that);
	// },
	onShow: function() {
		let that = this;
		that.page = 1;
		that.isLastPage = false;
		that.loadPageData(that);
	},
	onReachBottom: function() {
		let that = this;
		if (that.isLastPage) {
			return;
		}
		that.page = that.page + 1;
		that.loadPageData(that);
	}
};
</script>

<style lang="less">
@import url('message_list.less');
</style>
