<template lang="html">
	<section class="goods">
		<div class="menu-wrapper" ref="menuWrapper">
			<ul>
				<li v-for="(item,index) in goods" class="menu-item menu-item-hook" :class="{'current':index==currentIndex}" @click="selectMenu(index,$event)">
					<span class="text border-1px">
						<span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
						{{item.name}}
					</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" ref="foodsWrapper">
			<ul>
				<li v-for="item in goods" class="foods-list food-list-hook">
					<h1 class="title" v-text="item.name"></h1>
					<ul>
						<li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px">
							<div class="icon">
								<img :src="food.icon" width="57" height="57">
							</div>
							<div class="content">
								<h2 class="name" v-text="food.name"></h2>
								<p class="desc" v-text="food.description"></p>
								<div class="extra">
									<span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
								</div>
								<div class="price">
									<span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
								</div>
								<div class="cartControl-wrapper">
									<cartControl :food="food" @cartAdd="cartAdd"></cartControl>
								</div>
							</div>
						</li>
					</ul>
				</li>
			</ul>
		</div>
		<shoppingCart ref='shoppingCart' :select-foods='selectFoods' :delivery-price='seller.deliveryPrice' :min-price='seller.minPrice'></shoppingCart>
		<food :food="selectedFood" ref="food" @cartAdd="cartAdd"></food>
	</section>
</template>

<script>
	import axios from 'axios'
	import BScroll from 'better-scroll'
	import shoppingCart from '../shoppingCart/shoppingCart'
	import cartControl from '../cartControl/cartControl'
	import food from '../food/food'

	const ERR_OK = 0

	export default {
		props: {
			seller: {
				type: Object
			}
		},
		components: {
			shoppingCart, cartControl, food
		},
		data () {
			return {
				goods: [],
				listHeight: [],
				scrollY: 0,
				selectedFood: {}
			}
		},
		created () {
			let _this = this
			axios.get('/api/goods').then((res) => {
        if (res.data.errno === ERR_OK) {
          _this.goods = res.data.data
					_this.$nextTick(() => {
						_this.initScroll()
						_this.caculateHeight()
					})
        }
      })
			this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
		},
		computed: {
			currentIndex() {
				for (let i = 0; i < this.listHeight.length; i++) {
					let height1 = this.listHeight[i]
					let height2 = this.listHeight[i + 1]
					if (!height2 || this.scrollY >= height1 && this.scrollY < height2) {
						return i
					}
				}
				return 0
			},
			selectFoods() {
				let foods = []
				this.goods.forEach((good) => {
					good.foods.forEach((food) => {
						if (food.count) {
							foods.push(food)
						}
					})
				})
				return foods
			}
		},
		methods: {
				initScroll() {
					this.menuScroll = new BScroll(this.$refs.menuWrapper, {
						click: true
					})
					this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
						click: true,
						probeType: 3
					})
					this.foodsScroll.on('scroll', (pos) => {
						this.scrollY = Math.abs(Math.round(pos.y))
						let menuItem = this.$refs.menuWrapper.getElementsByClassName('menu-item-hook')
						let el = menuItem[this.currentIndex]
						this.menuScroll.scrollToElement(el, 500)
					})
				},
				// 计算右边栏的每个区域的高度
				caculateHeight() {
					let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
					let height = 0
					this.listHeight.push(height)
					for (let i = 0; i < foodList.length; i++) {
						let item = foodList[i]
						height += item.clientHeight
						this.listHeight.push(height)
					}
				},
				selectMenu(index, e) {
					if (!e._constructed) {
						return
					}
					let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
					let el = foodList[index]
					this.foodsScroll.scrollToElement(el, 500)
				},
				cartAdd(target) {
					this.$refs.shoppingCart.drop(target)
				},
				selectFood(food, e) {
					if (!e._constructed) {
						return
					}
					this.selectedFood = food
					this.$refs.food.show()
				}
		}
	}
</script>

<style lang="stylus" scoped>
	@import "../../common/style/mixin.styl"

	.goods {
		display: flex;
		position: absolute;
		top: 174px;
		bottom: 46px;
		width: 100%;
		overflow: hidden;
		.menu-wrapper {
			flex: 0 0 80px;
			width: 80px;
			background: #f3f5f7;
			.menu-item {
				display: table;
				height: 54px;
				width: 100%;
				line-height: 14px;
				&.current {
					position: relative;
					z-index: 10;
					margin-top: -1px;
					background-color: #fff;
					font-weight: 700;
					.text {
						border-none();
					}
				}
				.icon {
					display: inline-block;
					vertical-align: top;
					width: 12px;
					height: 12px;
					margin-right: 2px;
					background-size: 100% 100%;
					background-repeat: no-repeat;
					&.decrease {
						bg-image('decrease_3');
					}
					&.discount {
						bg-image('discount_3');
					}
					&.guarantee {
						bg-image('guarantee_3');
					}
					&.invoice {
						bg-image('invoice_3');
					}
					&.special {
						bg-image('special_3');
					}
				}
				.text {
					display: table-cell;
					width: 56px;
					vertical-align: middle;
					font-size: 12px;
					padding: 0 14px;
					box-sizing: border-box;
					border-1px(rgba(1, 17, 27, 0.1));
				}
			}
		}
		.foods-wrapper {
			flex: 1;
			.title {
				padding-left: 14px;
				height: 26px;
				line-height: 26px;
				border-left: 2px solid #d9dde1;
				font-size: 12px;
				color: rgb(147, 153, 159);
				background-color: #f3f5f7;
			}
			.food-item {
				display: flex;
				margin: 18px;
				padding-bottom: 18px;
				border-1px(rgba(1, 17, 27, 0.1));
				&:last-child {
					border-none();
					margin-bottom: 0;
				}
				.icon {
					flex:0 0 57px;
					margin-right: 10px;
				}
				.content {
					flex: 1;
					.name {
						margin: 2px 0 8px;
						height: 14px;
						line-height: 14px;
						font-size: 10px;
						color: rgb(7, 17, 27);
					}
					.desc, .extra {
						line-height: 14px;
						font-size: 10px;
						color: rgb(147, 153, 159);
					}
					.desc {
						margin-bottom: 8px;
					}
					.extra {
						.count {
							margin-right: 12px;
						}
					}
					.price {
						font-weight: 700;
						line-height: 24px;
						.now {
							margin-right: 8px;
							font-size: 14px;
							color: rgb(240, 20, 20);
						}
						.old {
							text-decoration: line-through;
							font-size: 10px;
							color: rgb(147, 153, 159);
						}
					}
					.cartControl-wrapper {
						position: absolute;
						right: 0;
						bottom: 10px;
					}
				}
			}
		}

	}
</style>
