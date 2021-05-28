<template>
	<span class="point chart__point"
		:style="setStyleNumberPoint"
	>
		{{ nPoint }} ({{ setStyleCoordPoints }})
	</span>
</template>

<script>
export default {
	name: 'cPoint',
	props: {
		primaryCoords: {
			type: Object,
			default: () => ({})
		},
		filling: {
			type: Array,
			default: () => ([])
		},
		points: {
			type: Object,
			default: () => ({})
		},
		nPoint: {
			type: [Number, String],
			default: 0
		},
		isEnd: {
			type: Boolean,
			default: false
		}
	},
	computed: {
		setStyleNumberPoint() {
			const {x, y} = this.points
			// const res = this.filling.map((curr, i) => {
			// 	return this.filling.includes(curr)
			// })
			// console.log({res})
			// console.log('fill', this.filling, JSON.stringify(this.points))
			// const res = this.filling
			// 	.map(curr => curr === JSON.stringify(this.points))
			// 	// .filter(curr => curr === false)
			// console.log(res)
			return {
				left: `${x + 15}px`,
				top: `${y - 15}px`
			}
		},
		setStyleCoordPoints() {
			return this.primaryCoords[this.nPoint]
		}
	},
	watch: {
		isEnd(val) {
			console.log('end', val, this.filling)
		}
	},
	created() {
		this.$emit('fillingCoords')
	}
}
</script>

<style lang="scss">
	.chart {
		&__point {
			position: absolute;
		}
	}
	.point {
		font-size: 1.8rem
	}
</style>