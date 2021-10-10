<template>
	<view class="detail">
		<view class="fixbg" :style="{backgroundImage:'url('+ songDetail.al.picUrl +')'}"></view>
		<musichead :title="songDetail.name" :icon="true" color="white"></musichead>
		<view class="container" v-show="!isLoading">
			<scroll-view scroll-y="true">
				<!--播放转盘-->
				<view class="detail-play" @tap="handleToPlay">
					<image :src="songDetail.al.picUrl" :class="{ 'detail-play-run' : isPlayRotate }" mode=""></image>
		        		<text class="iconfont" :class="iconPlay"></text>
					<view></view>
				</view>
				<!--歌词-->
				<view class="detail-lyric">
					<view class="detail-lyric-wrap"
						:style="{ transform : 'translateY(' +  - (lyricIndex - 1) * 82  + 'rpx)' }">
						<view class="detail-lyric-item" :class="{ active : lyricIndex == index}"
							v-for="(item,index) in songLyric" :key="index">{{ item.lyric }}</view>
					</view>
				</view>
				<!--喜欢这首歌的人也听-->
				<view class="detail-like">
					<view class="detail-like-head">喜欢这首歌的人也听</view>
					<view class="detail-like-item" v-for="(item,index) in songSimi" :key="index"
						@tap="handleToSimi(item.id)">
						<view class="detail-like-img">
							<image :src="item.album.picUrl" mode=""></image>
						</view>
						<view class="detail-like-song">
							<view>{{ item.namae }}</view>
							<view>
								<image src="../../static/dujia.png" mode=""></image>
								<image v-if="item.privilege.maxbr == 999000" src="../../static/sq.png" mode=""></image>
								{{ item.album.artists[0].name }} - {{ item.name }}
							</view>
						</view>
						<text class="iconfont iconbofang"></text>
					</view>
				</view>
				<!-- 精彩评论 -->
				<view class="detail-comment">
					<view class="detail-comment-head">精彩评论</view>
					<view class="detail-comment-item" v-for="(item,index) in songComment" :key="index">
						<view class="detail-comment-img">
							<image :src="item.user.avatarUrl" mode=""></image>
						</view>
						<view class="detail-comment-content">
							<view class="detail-comment-title">
								<view class="detail-comment-name">
									<view>{{ item.user.nickname }}</view>
									<view>{{ item.time | formatTime }}</view>
								</view>
								<view class="detail-comment-like">{{ item.likedCount | formatCount }}<text
										class="iconfont iconlike"></text></view>
							</view>
							<view class="detail-comment-text">{{ item.content }}</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import musichead from '../../components/musichead/musichead.vue'
	import '@/common/iconfont.css';
	import {
		songDetail,
		songSimi,
		songComment,
		songLyric,
		songUrl
	} from '../../common/api.js'

	export default {
		data() {
			return {
				songDetail: {
					al: {
						picUrl: ''
					},
					ar: {
						name: ''
					}
				},
				songSimi: [],
				songComment: [],
				songLyric: [],
				lyricIndex: 0,
				iconPlay: 'iconpause',
				isPlayRotate: true,
				isLoading: true,
			}
		},
		components: {
			musichead
		},
		// 接收传递过来的歌曲id
		onLoad(options) {
			// console.log(options.songId)
			this.getMusic(options.songId);
		},
		// 离开当前页面，回到上一级的时候，清除定时器
		onUnload() {
			this.cancelLyricIndex()
			// #ifdef H5
			this.bgAudioManager.destroy()
			// #endif
		},
		// 回到首页的时候，清除定时器
		onHide() {
			this.cancelLyricIndex()
			// #ifdef H5
			this.bgAudioManager.destroy()
			// #endif
		},
		methods: {
			getMusic(songId) {this.$store.commit('NEXT_ID', songId)
			uni.showLoading({title: '加载中...'});
				this.isLoading = true;

				Promise.all([songDetail(songId), songSimi(songId), songComment(songId), songLyric(songId), songUrl(songId)]).then((res) => {
					if (res[0][1].data.code == '200') {
						this.songDetail = res[0][1].data.songs[0];
					}
					if (res[1][1].data.code == '200') {
						this.songSimi = res[1][1].data.songs;
					}
					if (res[2][1].data.code == '200') {
						this.songComment = res[2][1].data.hotComments;
					}
					// 歌词
					if (res[3][1].data.code == '200') {
						// 先拿到歌词
						let lyric = res[3][1].data.lrc.lyric;
						// 正则表达式分隔歌词
						let re = /\[([^\]]+)\]([^[]+)/g;
						// 歌词停留的时间
						let result = [];
						lyric.replace(re, ($0, $1, $2) => {
							result.push({
								"time": this.formatTimeToSec($1),
								"lyric": $2
							});
						});
						// 进行映射
						this.songLyric = result;
					}
					// 音频地址
					if (res[4][1].data.code == '200') {

						// #ifdef MP-WEIXIN
						this.bgAudioManager = uni.getBackgroundAudioManager();
						this.bgAudioManager.title = this.songDetail.name;
						// #endif

						// #ifdef H5
						if (!this.bgAudioManager) {
							this.bgAudioManager = uni.createInnerAudioContext();
						}
						this.iconPlay = 'iconbofang1';
						this.isPlayRotate = false;
						// #endif

						this.bgAudioManager.src = res[4][1].data.data[0].url || '';
						this.listenLyricIndex();
						// 监听播放
						this.bgAudioManager.onPlay(() => {
							this.iconPlay = 'iconpause';
							this.isPlayRotate = true;
							this.listenLyricIndex();
						})
						// 监听暂停
						this.bgAudioManager.onPause(() => {
							this.iconPlay = 'iconbofang1';
							this.isPlayRotate = false;
							this.cancelLyricIndex();
						});
						// 播放完毕跳转到下一首
						this.bgAudioManager.onEnded(() => {
							this.getMusic(this.$store.state.naxtId);
						})
					}
					this.isLoading = false;
					uni.hideLoading();
				});
			},
			// 转化成秒
			formatTimeToSec(time) {
				// 分钟和秒分隔开后存放到数组中
				var arr = time.split(':');
				// 先把数字进行操作，再进行toFixed转换，最后返回转换成秒的结果
				return (parseFloat(arr[0]) * 60 + parseFloat(arr[1])).toFixed(1);
			},
			// 监听点击播放
			handleToPlay() {
				// 如果是播放状态就开始播放
				if (this.bgAudioManager.paused) {
					this.bgAudioManager.play();
				} else { // 否则暂停播放
					this.bgAudioManager.pause();
				}
			},
			//  利用节流实现监听事件慢加载
			listenLyricIndex() {
				clearInterval(this.timer);
				// 监听歌词的变化,500毫秒监听一次
				this.timer = setInterval(() => {
					for (var i = 0; i < this.songLyric.length; i++) {
						if (this.bgAudioManager.currentTime > this.songLyric[this.songLyric.length - 1].time) {
							this.lyricIndex = this.songLyric.length - 1;
							break;
						}
						if (this.bgAudioManager.currentTime > this.songLyric[i].time && this.bgAudioManager
							.currentTime < this.songLyric[i + 1].time) {
							this.lyricIndex = i;
						}
					}
					// console.log(this.lyricIndex);
				}, 500);
			},
			// 优化性能，防止暂停的时候定时器继续，影响性能
			cancelLyricIndex() {
				clearInterval(this.timer);
			},
			// 跳转切换歌曲，拿到handleToSimi.id 更新整个数据
			handleToSimi(songId) {
				this.getMusic(songId);
			}
		},
	}
</script>

<style scoped>
	/* 圆转盘 */
	.detail-play {
		width: 580rpx;
		height: 580rpx;
		background: url(~@/static/disc.png);
		background-size: cover;
		margin: 214rpx auto 0 auto;
		position: relative;
	}

	.detail-play image {
		width: 370rpx;
		height: 370rpx;
		border-radius: 50%;
		position: absolute;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		margin: auto;
		animation: 10s linear move infinite;
		animation-play-state: paused;
	}

	/* 播放旋转图片 */
	.detail-play .detail-play-run {
		animation-play-state: running;
	}

	@keyframes move {
		from {
			transform: rotate(0deg);
		}

		to {
			transform: rotate(360deg);
		}
	}

	.detail-play text {
		width: 100rpx;
		height: 100rpx;
		font-size: 100rpx;
		color: white;
		position: absolute;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		margin: auto;
	}

	.detail-play view {
		width: 230rpx;
		height: 360rpx;
		background: url(~@/static/needle.png);
		position: absolute;
		left: 100rpx;
		right: 0;
		top: -200rpx;
		margin: auto;
		background-size: cover;
	}

	/* 文字区域 */
	.detail-lyric {
		font-size: 32rpx;
		line-height: 82rpx;
		height: 246rpx;
		text-align: center;
		overflow: hidden;
		color: #6f6e73;
	}

	.detail-lyric-wrap {
		transition: .5s;
	}

	.detail-lyric-item {
		height: 82rpx;
	}

	.detail-lyric-item.active {
		color: white;
	}

	/* <!--喜欢这首歌的人也听--> */
	.detail-like {
		margin: 0 30rpx;
	}

	.detail-like-head {
		font-size: 36rpx;
		color: white;
		margin: 50rpx 0;
	}

	.detail-like-item {
		display: flex;
		align-items: center;
		margin-bottom: 28rpx;
	}

	.detail-like-img {
		width: 82rpx;
		height: 82rpx;
		border-radius: 20rpx;
		overflow: hidden;
		margin-right: 20rpx;
	}

	.detail-like-img image {
		width: 100%;
		height: 100%;
	}

	.detail-like-song {
		flex: 1;
		color: #c6c2bf;
	}

	.detail-like-song view:nth-child(1) {
		font-size: 28rpx;
		color: white;
		margin-bottom: 12rpx;
	}

	.detail-like-song view:nth-child(2) {
		font-size: 22rxp;
	}

	.detail-like-song image {
		width: 26rpx;
		height: 20rpx;
		margin-right: 10rpx;
	}

	.detail-like-song text {
		font-size: 50rpx;
		color: #C6C2BF;
	}

	/* 精彩评论 */
	.detail-comment {
		margin: 0 30rpx;
	}

	.detail-comment-head {
		font-size: 36rpx;
		color: white;
		margin: 50rpx 0;
	}

	.detail-comment-item {
		margin-bottom: 28rpx;
		display: flex;
	}

	.detail-comment-img {
		width: 64rpx;
		height: 64rpx;
		border-radius: 50%;
		overflow: hidden;
		margin-right: 18rpx;
	}

	.detail-comment-img image {
		width: 100%;
		height: 100%;
	}

	.detail-comment-content {
		flex: 1;
		color: #cbcacf;
	}

	.detail-comment-title {
		display: flex;
		justify-content: space-between;
	}

	.detail-comment-name {}

	.detail-comment-name view:nth-child(1) {
		font-size: 26rpx;
	}

	.detail-comment-name view:nth-child(2) {
		font-size: 20rpx;
	}

	.detail-comment-like {
		font-size: 28rpx;
	}

	.detail-comment-text {
		font-size: 28rpx;
		line-height: 40rpx;
		color: white;
		margin-top: 20rpx;
		border-bottom: 1px solid #e0e0e0;
		padding-bottom: 40rpx;
	}
</style>
