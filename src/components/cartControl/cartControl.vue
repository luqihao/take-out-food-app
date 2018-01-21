<template lang="html">
	<section class="cartControl">
		<transition name='move'>
			<div class="cart-decrease icon-remove_circle_outline" v-show="food.count>0" @click.stop="decreaseCart($event)">
			</div>
		</transition>
		<div class="cart-count" v-if="food.count>0" v-text="food.count">

		</div>
		<div class="cart-increase icon-add_circle" @click.stop="increaseCart($event)">

		</div>
	</section>
</template>

<script>
import Vue from 'vue'

export default {
	props: {
		food: {
			type: Object
		}
	},
	methods: {
		increaseCart(e) {
			if (!e._constructed) {
				return
			}
			if (!this.food.count) {
				Vue.set(this.food, 'count', 1)
			} else {
				this.food.count++
			}
			this.$emit('cartAdd', e.target)
		},
		decreaseCart(e) {
			if (!e._constructed) {
				return
			}
			this.food.count--
		}
	}
}
</script>

<style lang="scss">
.move-enter-active, .move-leave-active {
	transition: all .4s linear;
}
.move-enter, .move-leave-to {
	opacity: 0;
	transform: translateX(24px) rotateZ(1020deg);
}


	.cartControl {
		font-size: 0;
		.cart-decrease{
			display: inline-block;
			vertical-align: top;
			padding: 6px;
			display: inline-block;
			line-height: 24px;
			font-size: 24px;
			color: rgb(0, 160, 220);
			.inner {
				transform: rotateZ(0);
			}
		}
		.cart-decrease {

		}
		.cart-count {
			display: inline-block;
			vertical-align: top;
			width: 12px;
			padding-top: 6px;
			line-height: 24px;
			text-align: center;
			font-size: 10px;
			color: rgb(147, 153, 159);
		}
		.cart-increase{
			position: relative;
			display: inline-block;
			vertical-align: top;
			padding: 6px;
			line-height: 24px;
			font-size: 24px;
			color: rgb(0, 160, 220);
			.cart-increase-active {
				position: absolute;
				z-index: -1;
				top: 0;
				left: 0;
				padding: 6px;
				line-height: 24px;
				font-size: 24px;
				color: rgb(100, 260, 220);
			}
		}
	}
</style>
