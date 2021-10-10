<template>
	<view class="index">
		<musichead title="网易云音乐" :icon="false"></musichead>
		<view class="container">
			<!-- scroll-y="true" 表示y轴可以进行滚动 -->
			<scroll-view scroll-y="true">
				<view class="index-search" @tap="handleToSearch">
					<text class="iconfont iconsearch"></text>
					<input type="text" placeholder="搜索歌曲" />
				</view>
				<view v-if="isLoading">
					<m-for-skeleton
					:avatarSize="200"
					:row="3"
					:loading="isLoading"
					:isarc="'square'"
					v-for="(item,key) in 4"
					:titleStyle="{}"
					:title="false"
					:key="key">
					</m-for-skeleton>
				</view>
				<!-- 图片歌曲模块 -->
				<view class="index-list" v-else>
					<!-- <view class="index-list-item">
						<view class="index-list-img">
							<image src="../../static/wangyiyunyinyue.png" mode=""></image>
							<text>每天更新</text>
						</view>
						<view class="index-list-text">
							<view>1.与我无关 - 啊岑</view>
							<view>1.与我无关 - 啊岑</view>
							<view>1.与我无关 - 啊岑</view>
						</view>
					</view> -->
					<view class="index-list-item" v-for="(item,index) in topList" :key="index" @tap="handleToList(item.listId)">
						<view class="index-list-img">
							<image :src="item.coverImgUrl" mode=""></image>
							<text>{{ item.updateFrequency }}</text>
						</view>
						<view class="index-list-text">
							<view v-for="(item,index) in item.tracks" :key="index">{{ index + 1 }}.{{ item.first }} - {{ item.second }}</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import '@/common/iconfont.css';
	import musichead from '../../components/musichead/musichead.vue'
	import { topList } from '../../common/api.js'
	import mForSkeleton from "../../components/m-for-skeleton/m-for-skeleton.vue";
	
	export default {
		data() {
			return {
				topList: [],
				isLoading: true,
			}
		},
		onLoad() {
			topList().then((res)=>{
				if(res.length) {
					setTimeout(()=>{
						this.topList = res;
						this.isLoading = false;
					},1000);
				}
			})
		},
		methods: {
			// 调转到list.vue播放页面
			handleToList(listId) {
				uni.navigateTo({
					url:`/pages/list/list?id=${listId}`
				});
			},
			// 跳转搜索歌曲
			handleToSearch(){
			    uni.navigateTo({
					url: '/pages/search/search'
				});
			},
		},
		components: {
			musichead,
			mForSkeleton,
		},
	}
</script>

<style>
	.index{}
	.index-search{ display: flex; align-items: center; height: 70rpx; margin: 70rpx 30rpx 30rpx 30rpx; background:#f7f7f7; border-radius: 50rpx; }
	.index-search text { font-size: 26rpx; margin-right: 26rpx; margin-left: 28rpx; }
	.index-search input { font-size: 28rpx; flex: 1; }
	/* 图片歌曲模块 */
	.index-list { margin: 0 30rpx; }
	.index-list-item { display: flex; margin-bottom: 34rpx; }
	.index-list-img { width: 212rpx; height: 212rpx; position: relative; border-radius: 30rpx; overflow: hidden; margin-right: 22rpx; }
	.index-list-img image { width: 100%; height: 100%; }
	.index-list-img text { position: absolute; left: 12rpx; bottom: 16rpx; color: white; font-size: 20rpx; }
	.index-list-text { font-size: 24rpx; line-height: 66rpx; width: 70vw; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
</style>
