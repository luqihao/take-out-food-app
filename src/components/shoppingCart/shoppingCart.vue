<template lang="html">
	<section>
		<div class="shoppingCart">
			<div class="content">
				<div class="content-left">
					<div class="logo-wrapper" @click="toggleList">
						<div class="logo" :class="{'highlight':totalCount>0}">
							<i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
						</div>
						<div class="num" v-if="totalCount>0" v-text="totalCount">

						</div>
					</div>
					<div class="price" v-text="'￥'+totalPrice" :class="{'highlight':totalPrice>0}"></div>
					<div class="desc" v-text="'另需配送费￥' + deliveryPrice"></div>
				</div>
				<div class="content-right" @click.stop="toPay">
					<div class="pay" v-text="payDesc" :class="payClass"></div>
				</div>
			</div>
			<div class="ball-container">
				<transition v-for="ball in balls" :key="ball" @before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter">
					<div class="ball" v-show="ball.show">
						<div class="inner inner-hook icon-add_circle"></div>
					</div>
				</transition>
			</div>
			<transition name="fold">
				<div class="shoppingCart-list" v-show="listShow">
					<div class="list-header">
						<h1 class="title">购物车</h1>
						<span class="empty" @click="empty">清空</span>
					</div>
					<div class="list-content" ref="listContent">
						<ul>
							<li class="food" v-for='food in selectFoods'>
								<span class="name">{{food.name}}</span>
								<div class="price">
									<span>￥{{food.price*food.count}}</span>
								</div>
								<div class="cartControl-wrapper">
									<cartControl :food="food" @cartAdd="cartAdd"></cartControl>
								</div>
							</li>
						</ul>
					</div>
				</div>
			</transition>
		</div>
		<transition name="fade">
			<div class="list-mask" v-show="listShow" @click="fold = true">
			</div>
		</transition>
	</section>
</template>

<script>
import cartControl from '../cartControl/cartControl'
import BScroll from 'better-scroll'

export default {
	components: {
		cartControl
	},
	props: {
		selectFoods: {
			type: Array,
			default () {
				return []
			}
		},
		deliveryPrice: {
			type: Number,
			default: 0
		},
		minPrice: {
			type: Number,
			default: 0
		}
	},
	data() {
		return {
			balls: [
				{
					show: false
				},
				{
					show: false
				},
				{
					show: false
				}
			],
			dropBalls: [],
			fold: true
		}
	},
	computed: {
		totalPrice() {
			let total = 0
			this.selectFoods.forEach((food) => {
				total += food.price * food.count
			})
			return total
		},
		totalCount() {
			let count = 0
			this.selectFoods.forEach((food) => {
				count += food.count
			})
			return count
		},
		payDesc() {
			if (this.totalPrice === 0) {
				return '￥' + this.totalPrice + '元起送'
			} else if (this.totalPrice < this.minPrice) {
				return '还差￥' + (this.minPrice - this.totalPrice) + '元起送'
			} else {
				return '去结算'
			}
		},
		payClass() {
			if (this.totalPrice < this.minPrice) {
				return 'not-enough'
			} else {
				return 'enough'
			}
		},
		listShow() {
			if (!this.totalCount) {
				this.fold = true
				return false
			}
			let show = !this.fold
			if (show) {
				this.$nextTick(() => {
					if (!this.scroll) {
						this.scroll = new BScroll(this.$refs.listContent, {
							click: true
						})
					} else {
						this.scroll.refresh()
					}
				})
			}
			return show
		}
	},
	methods: {
		toPay() {
			if (this.totalPrice < this.minPrice) {
				return
			}
			alert('需支付￥' + (this.totalPrice + this.deliveryPrice))
		},
		empty() {
			this.selectFoods.forEach((food) => {
				food.count = 0
			})
		},
		cartAdd(target) {
			this.drop(target)
		},
		toggleList() {
			if (!this.totalCount) {
				return
			}
			this.fold = !this.fold
		},
    drop(el) {
    // 触发一次事件就会将所有小球进行遍历
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i]
        if (!ball.show) { // 将false的小球放到dropBalls
          ball.show = true
          ball.el = el // 设置小球的el属性为一个dom对象
          this.dropBalls.push(ball)
          return
        }
      }
    },
		// 这个方法的执行是因为这是一个vue的监听事件
    beforeEnter(el) {
      let count = this.balls.length
      while (count--) {
        let ball = this.balls[count]
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect() // 获取小球的相对于视口的位移(小球高度)
          let x = rect.left - 32
          let y = -(window.innerHeight - rect.top - 22) // 负数,因为是从左上角往下的的方向
          el.style.display = '' // 清空display
          el.style.webkitTransform = `translate3d(0,${y}px,0)`
          el.style.transform = `translate3d(0,${y}px,0)`
          // 处理内层动画
          let inner = el.getElementsByClassName('inner-hook')[0] // 使用inner-hook类来单纯被js操作
          inner.style.webkitTransform = `translate3d(${x}px,0,0)`
          inner.style.transform = `translate3d(${x}px,0,0)`
        }
      }
    },
		// 这个方法的执行是因为这是一个vue的监听事件
    enter(el, done) {
      /* eslint-disable no-unused-vars */
      let rf = el.offsetHeight // 触发重绘html
      this.$nextTick(() => { // 让动画效果异步执行,提高性能
        el.style.webkitTransform = 'translate3d(0,0,0)'
        el.style.transform = 'translate3d(0,0,0)'
        // 处理内层动画
        let inner = el.getElementsByClassName('inner-hook')[0] // 使用inner-hook类来单纯被js操作
        inner.style.webkitTransform = 'translate3d(0,0,0)'
        inner.style.transform = 'translate3d(0,0,0)'
        el.addEventListener('transitionend', done) // Vue为了知道过渡的完成，必须设置相应的事件监听器。
      })
    },
		// 这个方法的执行是因为这是一个vue的监听事件
    afterEnter(el) {
      let ball = this.dropBalls.shift() // 完成一次动画就删除一个dropBalls的小球
      if (ball) {
        ball.show = false
        el.style.display = 'none' // 隐藏小球
      }
    }
  }
}
</script>

<style lang="stylus">
@import "../../common/style/mixin.styl"
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to {
  opacity: 0
}

	.shoppingCart {
		position: fixed;
		left: 0;
		bottom:0;
		z-index: 50;
		width: 100%;
		height: 48px;
		.content {
			display: flex;
			background-color: #141d27;
			font-size: 0;
			.content-left {
				flex: 1;
				.logo-wrapper {
					display: inline-block;
					vertical-align: top;
					position: relative;
					top: -10px;
					margin: 0 12px;
					padding: 6px;
					width: 56px;
					height: 56px;
					box-sizing: border-box;
					border-radius: 50%;
					background-color: #141d27;
					.logo {
						width:100%;
						height: 100%;
						border-radius: 50%;
						background-color: #2b343c;
						text-align: center;
						&.highlight {
							background-color: rgb(0, 160, 220);
						}
						.icon-shopping_cart {
							line-height: 44px;
							font-size: 24px;
							color: #80858a;
							&.highlight {
								color: #fff;
							}
						}
					}
					.num {
						position: absolute;
						top: 0;
						right: 0;
						width: 24px;
						height: 16px;
						line-height: 16px;
						text-align: center;
						border-radius: 16px;
						font-size: 9px;
						font-weight: 700;
						color: #fff;
						background-color: rgb(240, 20, 20);
						box-shadow: 0 4px 8px 0 rgba(0, 0, 0, .4);
					}
				}
				.price {
					display: inline-block;
					vertical-align: top;
					margin-top: 12px;
					line-height: 24px;
					padding-right: 12px;
					box-sizing: border-box;
					border-right: 1px solid rgba(255, 255, 255, .1);
					font-size: 16px;
					font-weight: 700;
					color:rgba(255, 255, 255, .4);
					&.highlight {
						color: #fff;
					}
				}
				.desc {
					display: inline-block;
					vertical-align: top;
					line-height: 24px;
					margin: 12px 0 0 12px;
					line-height: 24px;
					font-size: 14px;
					color: rgba(255, 255, 255, .4);
				}
			}
			.content-right {
				flex: 0 0 105px;
				width: 105px;
				.pay {
					height: 48px;
					line-height: 48px;
					text-align: center;
					font-size: 12px;
					font-weight: 700;
					color: rgba(255, 255, 255, .4);
					background-color: #2b343c;
					&.not-enough {
						background-color: #2b343c;
					}
					&.enough {
						background-color: rgb(0, 160, 220);
						color: #fff;
					}
				}
			}
		}
		.ball-container {
			.ball {
				position: fixed;
				left: 32px;
				bottom: 22px;
				z-index: 200;
				transition: all .4s cubic-bezier(0.49, -0.29, 0.75, 0.41);
        .inner {
					color: rgb(0,160,220);
          transition: all .4s linear;
				}
			}
		}
		.shoppingCart-list {
			position: absolute;
			top: 0;
			left: 0;
			z-index: -1;
			width: 100%;
			transform: translate3d(0, -100%, 0);
			transition: all .4s;
			&.fold-enter, &.fold-leave-to {
				transform: translate3d(0, 0, 0);
			}
			.list-header {
				height: 40px;
				line-height: 40px;
				padding: 0 18px;
				background: #141d27;
				border-bottom: 1px solid rgba(7, 17, 27, 0.1);
				color: rgb(255, 255, 255);
				.title {
					float: left;
					font-size: 14px;
				}
				.empty {
					float: right;
					font-size: 12px;
					color: rgb(0,160,220);
				}
			}
			.list-content {
				padding: 0 18px;
				max-height: 271px;
				overflow: hidden;
				background: #fff;
				.food {
					position: relative;
					padding: 12px 0;
					box-sizing: border-box;
					border-1px(rgba(7, 17, 27, 0.1));
					.name {
						line-height: 24px;
						font-size: 14px;
						color: rgb(7, 17, 27);
					}
					.price {
						position: absolute;
						right: 90px;
						bottom: 12px;
						line-height: 24px;
						font-size: 14px;
						font-weight: 700;
						color: rgb(240, 20, 20);
					}
					.cartControl-wrapper {
						position: absolute;
						right: -8px;
						bottom: 6px;
					}
				}
			}
		}
	}
	.list-mask {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 40;
		background: rgba(7, 17, 27, 0.6);
	}
</style>
