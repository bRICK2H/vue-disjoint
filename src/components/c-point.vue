<template>
	<span class="point chart__point"
		:style="setStyleNumberPoint"
	>
		{{ keyPoint }} ({{ points }})
	</span>
</template>

<script>
export default {
	name: 'cPoint',
	props: {
		point: {
			type: Object,
			default: () => ({})
		},
		keyPoint: {
			type: [Number, String],
			default: 0
		},
		max: {
			type: Number,
			default: 0
		},
		scale: {
			type: Number,
			default: 0
		}
	},
	computed: {
		setStyleNumberPoint() {
			const { x, y } = this.point

			return {
				left: `${this.scalable(x) + 15}px`,
				top: `${this.scalable(y) - 15}px`
			}
		},
		points() {
			const { x, y } = this.point

			return `${x}, ${y}`
		}
	},
	methods: {
		scalable(point) {
			return (point * this.scale) / this.max
		}
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
		transition: .5s;
		font-size: 1.8rem;
		font-weight: 400;
		white-space: pre;
	}
</style>