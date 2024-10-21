<template>
	<view>
		<image class="homeBg" src="../../static/image/home/homeBg.png" mode="aspectFill"></image>
		<view class="centerBox">
			<scroll-view scroll-y="true" style="width: 100%;height: 100%;">
				<view class="content flex align-center justify-center">
					<view class="content-box">
						<view class=" flex align-center justify-center">
							<image
								style="transition: left  0.5s ease-in-out,top 0.5s ease-in-out,transform 0.5s ease-in-out;"
								:style="{left:-womanLeft+'rpx',top:womanTop+'rpx',scale: scaleValues}" class="woman"
								src="../../static/image/my/woman.png" mode="">
							</image>
							<image
								style="transition: left  0.5s ease-in-out,top 0.5s ease-in-out,transform 0.5s ease-in-out;"
								:style="{left:-manLeft+'rpx',top:manTop+'rpx',scale: scaleValue}" class="man"
								src="../../static/image/my/man.png" mode="">
							</image>


						</view>
						<view class="content-box-item flex align-center justify-between flex-wrap"
							v-for="(item,index) in checkerboard" :key="index">
							<view class="content-box-itemz" v-for="(ite,ind) in item.tdList" :key="ind">
								<block v-if="index == 0 && ind === 5">
									<view class="flex align-center justify-center" style="width: 100%;height: 100%;">
										<image style="width: 70rpx;height: 70rpx;"
											src="../../static/image/my/startGames.png" mode="">
										</image>
									</view>

								</block>
								<block v-else>
									<view class="content-box-items flex align-center justify-center"
										:style="{background:ite.isShow?ite.color:'transparent'}">
										<block v-if="ite.isShow">
											<!-- <image style="width: 50rpx;height: 56rpx;" v-if="index == 0 && ind == 5"
												src="../../static/image/my/startGames.png" mode=""></image> -->
											<image style="width: 50rpx;height: 56rpx;"
												v-if="getIndexs()[0] == index && getIndexs()[1] == ind"
												src="../../static/image/home/endAdd.png" mode=""></image>
											<u-icon v-else name="heart-fill" color="rgba(255,255,255,0.5)"
												size="28"></u-icon>
										</block>
									</view>
									<view :style="{background:ite.isShow?ite.color2:'transparent'}"
										class="content-box-items2">
									</view>
								</block>

							</view>
						</view>


					</view>
				</view>
			</scroll-view>

		</view>
		<view class="bottom">
			<view class="szc flex align-center justify-center">
				<view class="sz flex align-center justify-center flex-wrap" @tap.stop="RollDice()">
					<image v-if="isDicing" :src="diceAnimationImages[aniIndex]"></image>
					<image v-else :src="gameIcon[currentIndex]"></image>
					<view style="background-color: #8247FB;" class="sz-sex" v-if="!isSex">
						男生掷
					</view>
					<view style="background-color: #FF8DEC;" class="sz-sex" v-else>
						女生掷
					</view>
				</view>
			</view>
		</view>


		<!-- 任务弹窗 -->
		<u-popup :show="show" bgColor="transparent" mode="center" @close="close">
			<view class="rw">
				<view class="rw-info">
					<!-- 男 -->
					<!-- <view class="rw-info-title flex align-center justify-center" v-if="!isSex">
						<u-icon style="margin-right: 10rpx;" name="man" color="#ffffff" size="28"></u-icon>
						男生
					</view>
					<view class="rw-info-title flex align-center justify-center" v-else>
						<u-icon style="margin-right: 10rpx;" name="woman" color="#ffffff" size="28"></u-icon>
						女生
					</view> -->
					<view class="rw-info-title flex align-center justify-center">
						任务
					</view>
					<view class="rw-info-content">
						{{content}}
					</view>

				</view>
			</view>
		</u-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				show: false,
				isDicing: false, //是否正在摇骰子
				gameIcon: [
					'/static/image/my/1dian.png',
					'/static/image/my/2dian.png',
					'/static/image/my/3dian.png',
					'/static/image/my/4dian.png',
					'/static/image/my/5dian.png',
					'/static/image/my/6dian.png'
				],
				diceAnimationImages: [
					'/static/image/my/dA.png',
					'/static/image/my/dB.png',
					'/static/image/my/dC.png',
					'/static/image/my/dD.png',
				],
				timer: null,
				aniIndex: 0,
				currentIndex: 0,
				isSex: false, //false:男 true:女
				bgColors: 'transparent',
				checkerboard: [], //棋盘
				colorsList: [{
					color1: '#FF75E8',
					color2: '#DF60C9',
				}, {
					color1: '#FF7F7E',
					color2: '#D56A68',
				}, {
					color1: '#FFD076',
					color2: '#D9B269',
				}, {
					color1: '#BF5FFF',
					color2: '#A555DB',
				}], //棋子颜色数组
				isShowArr: [], //需要显示的棋子
				showLists: [],
				coordinateMan: [0, 5], //男当坐标
				coordinateWoan: [0, 5], //女当坐标
				row: 9, //行
				tdList: 6, //列
				womanLeft: -580,
				womanTop: 40,
				manLeft: -618, //-24 120  238 356 474
				manTop: 40,
				scaleValue: 1, //原始大小
				scaleValues: 1, //原始大小
				isStart: true, //用作防抖
				gameTypeId: '', //游戏分类id
				content: '', //任务内容
				soundFilePath: '/static/audio/tantiao.mp3',
				soundAudioContent: null,
				soundFilePaths: '/static/audio/shaizi.mp3',
				soundAudioContents: null,
				adminMap: require('./index.json')
			};
		},
		onLoad(option) {
			console.log(this.adminMap)
			this.createAudio()
			this.getGame()
		},
		methods: {
			//游戏结束，初始化
			initGame() {
				//初始化人物位置
				this.womanLeft = -580
				this.womanTop = 40
				this.manLeft = -618
				this.manTop = 40
				//初始化人物坐标点
				this.coordinateMan = [0, 5] //男当坐标
				this.coordinateWoan = [0, 5] //女当坐标
				//初始化地图
				this.checkerboard.map((item, index) => {
					item.tdList.map((ite, ind) => {
						ite.isShow = ite.tdType === 1 ? true : false
						ite.manIsGo = ite.tdType === 1 ? false : true
						ite.womanIsGo = ite.tdType === 1 ? false : true
					})
				})
				this.isSex = false

			},
			//找出棋盘中最后一个显示棋子的位置
			getIndexs() {
				let indexs = -1
				let inds = -1
				this.checkerboard.map((item, index) => {
					item.tdList.map((ite, ind) => {
						if (ite.isShow == true) {
							indexs = index
							inds = ind
						}
					})
				})
				return [indexs, inds]
			},
			//创建声音
			createAudio() {
				//移动的声音
				this.soundAudioContent = uni.createInnerAudioContext()
				this.soundAudioContent.src = this.soundFilePath
				//掷骰子的声音
				this.soundAudioContents = uni.createInnerAudioContext()
				this.soundAudioContents.src = this.soundFilePaths
			},
			//初始化地图，并设置颜色
			setCOlor() {
				//棋子设置颜色
				if (this.checkerboard.length > 0) {
					this.checkerboard.map((item, index) => {
						item.tdList.map((ite, ind) => {
							if (index == 0 && ind == 5) {
								ite.manIsGo = true
								ite.womanIsGo = true
							}
							let indexss = this.mathRodamCOlor(index, ind)
							ite.color = this.colorsList[indexss].color1
							ite.color2 = this.colorsList[indexss].color2
						})
					})
				}
			},
			//获取游戏地图
			getGame() {
				// 从接口获取棋盘数据
				let adminMap = this.adminMap
				this.checkerboard = this.setCheckerboard(adminMap)
				if (this.checkerboard.length > 0) {
					this.setCOlor()
				}

			},
			//掷骰子声音
			playSound() {
				this.soundAudioContents.play()
			},
			// 跳动声音
			playTiao() {
				this.soundAudioContent.play()
			},
			//关闭任务弹窗
			close() {
				this.isStart = true
				this.show = false
				this.currentIndex = 0
				this.aniIndex = 0
				// this.currentIndex = num - 1
				this.isSex = !this.isSex
				//游戏结束(只要有一个人到达终点就判定游戏结束)
				if (this.getEquals(this.coordinateMan, this.getIndexs()) || this.getEquals(
						this.coordinateWoan, this.getIndexs())) {
					let that = this
					uni.showModal({
						title: '提示',
						content: '游戏结束',
						showCancel: false,
						confirmText: '重新开始',
						complete(ret) {
							that.initGame()
						}
					})
				}
			},
			//开启动画效果
			async startAnimation() {
				return new Promise((resolve) => {
					//设置筛子开始运动
					this.isDicing = true
					this.playSound()
					//记录动画次数
					let num = 0
					//每隔200毫秒来回切换一张“动”图形成掷骰子的动画
					this.timer = setInterval(() => {
						let index = this.aniIndex
						index++
						if (index >= this.diceAnimationImages.length) {
							index = 0
						}
						this.aniIndex = index
						num++
						//差不多执行1.6秒钟的时候可以停止了
						if (num > 8) {
							//关闭定时器
							clearInterval(this.timer)
							//设置骰子停止
							this.isDicing = false
							//返回结果
							resolve(true)
						}
					}, 200)
				})
			},
			//获取任务
			getTask() {
				this.content = '这是从接口获取到的任务'
				this.show = true
			},
			// 掷骰子
			async RollDice() {
				if (this.isStart == true) {
					this.isStart = false
					//掷骰子的点数,也是需要跳的格子数量
					let num = Math.floor(Math.random() * 6) + 1
					// let num = 6
					this.currentIndex = num - 1
					//开启骰子动画
					await this.startAnimation()
					console.log(this.coordinateMan, '111111')
					if (this.isSex) {
						//女生移动
						this.moveDiceWMan(num)
					} else {
						//男生移动
						this.moveDiceMan(num)
					}
					let time = 1000
					if (num == 1) {
						time = 1000
					} else {
						time = 700
					}
					if (this.getEquals(this.coordinateMan, this.getIndexs()) || this.getEquals(
							this.coordinateWoan, this.getIndexs())) {

					} else {
						setTimeout(() => {
							this.getTask()
						}, time * num)
					}

				}
			},
			//判断两个数组是否相等
			getEquals(arr1, arr2) {
				let flag = false
				if (arr1.length != arr2.length) {
					flag = false
				} else {
					flag = arr1.every(val => arr2.includes(val)) && arr2.every(val => arr1.includes(val));
				}
				return flag
			},
			//移动棋子 roll:骰子点数
			moveDiceWMan(roll) {
				//移动棋子 roll为要移动的次数
				let delay = 0
				let time = 1000
				if (roll === 1) {
					time = 1000
				} else {
					time = 700
				}
				for (let i = 0; i < roll; i++) {
					let isOver = false
					//获取当前棋子位置
					let nowPiece = this.coordinateWoan
					let leftShow = true
					let rightShow = true
					let topShow = true
					if (this.checkerboard[nowPiece[0]].tdList[nowPiece[1] - 1] && this.checkerboard[nowPiece[0]].tdList[
							nowPiece[1] - 1].womanIsGo != true) {
						leftShow = this.checkerboard[nowPiece[0]].tdList[nowPiece[1] - 1].womanIsGo
					}
					if (this.checkerboard[nowPiece[0]].tdList[nowPiece[1] + 1] && this.checkerboard[nowPiece[0]].tdList[
							nowPiece[1] + 1].womanIsGo != true) {
						rightShow = this.checkerboard[nowPiece[0]].tdList[nowPiece[1] + 1].womanIsGo
					}
					if (this.checkerboard[nowPiece[0] + 1] && this.checkerboard[nowPiece[0] + 1].tdList[nowPiece[1]] &&
						this.checkerboard[nowPiece[0] + 1]
						.tdList[nowPiece[1]].womanIsGo != true) {
						topShow = this.checkerboard[nowPiece[0] + 1].tdList[nowPiece[1]].womanIsGo
					}
					//左边可以走
					if (leftShow == false && rightShow != false) {
						setTimeout(() => {
							if (nowPiece[0] == 0 && nowPiece[1] == 5) {
								this.womanLeft += 106
							} else {
								this.womanLeft += 118
							}
							this.playTiao()
							this.scaleValues = 1.4
							setTimeout(() => {
								this.scaleValues = 1
							}, time / 2)
						}, time * delay)
						this.coordinateWoan = [nowPiece[0], nowPiece[1] - 1]
						this.checkerboard[nowPiece[0]].tdList[nowPiece[1] - 1].womanIsGo = true
					} else if (leftShow != false && rightShow == false) {
						//右边可以走
						setTimeout(() => {
							this.womanLeft += -118
							this.playTiao()
							this.scaleValues = 1.4
							setTimeout(() => {
								this.scaleValues = 1
							}, time / 2)
						}, time * delay)
						this.coordinateWoan = [nowPiece[0], nowPiece[1] + 1]
						this.checkerboard[nowPiece[0]].tdList[nowPiece[1] + 1].womanIsGo = true
					} else if (leftShow != false && rightShow != false && topShow == false) {
						//走下边
						setTimeout(() => {
							this.womanTop += 120
							this.playTiao()
							this.scaleValues = 1.4
							setTimeout(() => {
								this.scaleValues = 1
							}, time / 2)
						}, time * delay)
						this.coordinateWoan = [nowPiece[0] + 1, nowPiece[1]]
						this.checkerboard[nowPiece[0] + 1].tdList[nowPiece[1]].womanIsGo = true
					} else {
						isOver = true
					}
					if (!isOver) {
						delay++;
					}

				}
				//已到终点，游戏结束
				setTimeout(() => {
					this.getTask()
				}, time * delay)
			},
			//移动棋子 roll:骰子点数
			moveDiceMan(roll) {
				let isOver = false
				//移动棋子 roll为要移动的次数
				let delay = 0
				let time = 1000
				if (roll === 1) {
					time = 1000
				} else {
					time = 700
				}
				for (let i = 0; i < roll; i++) {
					//获取当前棋子位置
					let nowPiece = this.coordinateMan
					let leftShow = true
					let rightShow = true
					let topShow = true
					if (this.checkerboard[nowPiece[0]].tdList[nowPiece[1] - 1] && this.checkerboard[nowPiece[0]].tdList[
							nowPiece[1] - 1].manIsGo != true) {
						leftShow = this.checkerboard[nowPiece[0]].tdList[nowPiece[1] - 1].manIsGo
					}
					if (this.checkerboard[nowPiece[0]].tdList[nowPiece[1] + 1] && this.checkerboard[nowPiece[0]].tdList[
							nowPiece[1] + 1].manIsGo != true) {
						rightShow = this.checkerboard[nowPiece[0]].tdList[nowPiece[1] + 1].manIsGo
					}
					if (this.checkerboard[nowPiece[0] + 1] && this.checkerboard[nowPiece[0] + 1].tdList[nowPiece[1]] &&
						this.checkerboard[nowPiece[0] + 1]
						.tdList[nowPiece[1]].manIsGo != true) {
						topShow = this.checkerboard[nowPiece[0] + 1].tdList[nowPiece[1]].manIsGo
					}
					//左边可以走
					if (leftShow == false && rightShow != false) {
						setTimeout(() => {
							if (nowPiece[0] == 0 && nowPiece[1] == 5) {
								this.manLeft += 144
							} else {
								this.manLeft += 118
							}
							this.playTiao()
							this.scaleValue = 1.4
							setTimeout(() => {
								this.scaleValue = 1
							}, time / 2)
						}, time * delay)

						this.coordinateMan = [nowPiece[0], nowPiece[1] - 1]
						this.checkerboard[nowPiece[0]].tdList[nowPiece[1] - 1].manIsGo = true
					} else if (leftShow != false && rightShow == false) {
						//右边可以走
						setTimeout(() => {
							this.manLeft += -118
							this.playTiao()
							this.scaleValue = 1.4
							setTimeout(() => {
								this.scaleValue = 1
							}, time / 2)
						}, time * delay)
						this.coordinateMan = [nowPiece[0], nowPiece[1] + 1]
						this.checkerboard[nowPiece[0]].tdList[nowPiece[1] + 1].manIsGo = true
					} else if (leftShow != false && rightShow != false && topShow == false) {
						//走下边
						setTimeout(() => {
							this.manTop += 120
							this.playTiao()
							this.scaleValue = 1.4
							setTimeout(() => {
								this.scaleValue = 1
							}, time / 2)
						}, time * delay)
						this.coordinateMan = [nowPiece[0] + 1, nowPiece[1]]
						this.checkerboard[nowPiece[0] + 1].tdList[nowPiece[1]].manIsGo = true
					} else {
						isOver = true
					}
					//如果到达终点delay不在继续累加
					if (!isOver) {
						delay++;
					}
				}
				if (isOver) {
					setTimeout(() => {
						this.getTask()
					}, time * delay)

				}
			},
			//判断棋子是否显示
			//随机棋子颜色
			mathRodamCOlor(i, j) {
				let randomNum = Math.floor(Math.random() * this.colorsList.length)
				let randomColor = this.colorsList[randomNum].color1;
				// 检查上方的棋子  
				if (i > 0 && this.checkerboard[i - 1] && this.checkerboard[i - 1].tdList[j] && this.checkerboard[
						i - 1]
					.tdList[j].color == randomColor) {
					return this.mathRodamCOlor(i, j);
				}
				// 检查左侧的棋子  
				if (j > 0 && this.checkerboard[i] && this.checkerboard[i].tdList[j - 1] && this.checkerboard[i]
					.tdList[j -
						1].color == randomColor) {
					return this.mathRodamCOlor(i, j);
				}
				// 没有发现颜色重复，返回随机颜色  
				return randomNum;
			},
			/**
			 * @param {Number} adminMap 管理端返回的棋盘
			 * 根据管理端返回的棋盘处理
			 */
			setCheckerboard(adminMap) {
				adminMap.map((item, index) => {
					item.tdList.map((ite, ind) => {
						ite.isShow = ite.tdType === 1 ? true : false
						ite.manIsGo = ite.tdType === 1 ? false : true
						ite.womanIsGo = ite.tdType === 1 ? false : true
					})
				})
				return adminMap
			},
		}
	}
</script>

<style lang="scss">
	.bottom {
		width: 100%;
		height: calc(100vh - 88vh);
		background: rgba(255, 255, 255, 0);
		position: fixed;
		bottom: 0;
	}

	.centerBox {
		width: 100%;
		height: 88vh;
		position: relative;
		overflow: hidden;

	}

	.szc {
		width: 100%;
		height: auto;
		position: relative;
		// margin-top: 50rpx;
	}

	.sz {
		width: 130rpx;
		height: 130rpx;
		border-radius: 50%;
		background: radial-gradient(circle, #FEE8EE 46%, #FFD0E9 100%);

		image {
			width: 90rpx;
			height: 90rpx;
			margin-top: 24rpx;
		}

		.sz-sex {
			border-radius: 24rpx;
			padding: 6rpx 16rpx;
			color: #ffffff;
			font-size: 26rpx;
		}
	}

	.rw {
		width: 580rpx;

		.rw-info {
			width: 100%;
			padding: 50rpx;
			background: linear-gradient(90deg, #f851ad 50%, #ff35d5 100%);
			border-radius: 30rpx;
		}

		.rw-info-title {
			width: 100%;
			// margin-top: 60rpx;
			font-size: 39rpx;
			font-weight: bold;
			color: #ffffff;
		}

		.rw-info-content {
			width: 100%;

			margin-top: 20rpx;
			color: #ffffff;
			font-size: 39rpx;
			text-align: center;
		}
	}

	.homeBg {
		width: 100%;
		height: 100vh;
		position: fixed;
		top: 0;
	}

	.content {
		width: 100%;
		margin-top: 30rpx;

		.content-box {
			width: 686rpx;
			position: relative;
		}

		.woman {
			width: 94rpx;
			height: 94rpx;
			position: absolute;
			top: 50%;
			left: -24rpx;
			transform: translate(0, -50%);
			z-index: 999;
		}

		.man {
			width: 94rpx;
			height: 94rpx;
			position: absolute;
			top: 50%;
			right: -24rpx;
			transform: translate(0, -50%);
			z-index: 999;
		}



		.content-box-item {
			width: 100%;
			position: relative;
		}

		.content-box-itemz {
			width: 94rpx;
			height: 110rpx;
			border-radius: 12rpx;
			margin-bottom: 10rpx;
			position: relative;

		}

		.content-box-items {
			width: 94rpx;
			height: 94rpx;
			border-radius: 12rpx;
			position: absolute;
			top: 0;
			left: 0;
			z-index: 99;


		}

		.content-box-items2 {
			position: absolute;
			width: 94rpx;
			height: 90rpx;
			border-radius: 12rpx;
			top: 16rpx;
			left: 50%;
			transform: translate(-50%, 0);
			z-index: 90;
		}
	}

	.content-box-itemsBos {
		box-shadow: 0 20rpx 40rpx rgba(0, 0, 0, 0.19), 0 12rpx 12rpx rgba(0, 0, 0, 0.23);
	}
</style>