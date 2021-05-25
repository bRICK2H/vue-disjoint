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
		points: {
			type: Object,
			default: () => ({})
		},
		nPoint: {
			type: [Number, String],
			default: 0
		}
	},
	computed: {
		setStyleNumberPoint() {
			const {x, y} = this.points
			return {
				left: `${x + 15}px`,
				top: `${y - 15}px`
			}
		},
		setStyleCoordPoints() {
			return this.primaryCoords[this.nPoint]
		}
	},
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