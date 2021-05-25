<template>
	<span class="segment chart__segment"
		:style="draw"
	></span>
</template>

<script>
export default {
	name: 'cSegment',
	props: {
		segments: {
			type: Array,
			default: () => ([])
		}
	},
	data: () => ({}),
	computed: {
		draw() {
			if (this.segments.includes(undefined)) return;

			const [{ x: x1, y: y1 }, { x: x2, y: y2 }] = this.segments
			const deg = +(180 / Math.PI * Math.atan2(y2 - y1, x2 - x1)).toFixed(0)
			// const sin = +Math.sin(deg * Math.PI / 180).toFixed(2)
			const isNegative = /-/.test(String(deg))

			let rx1, rx2, ry1, ry2 = 0

			if (deg) {
				rx1 = x1 + 2
				rx2 = x2 + 2

				if (isNegative) {
					if (deg < -90) {
						ry1 = y1 - 2
						ry2 = y2 - 2
					} else if (deg > -90) {
						ry1 = y1 - 2
						ry2 = y2 - 2
					} else {
						ry1 = y1
						ry2 = y2
					}
				} else {
					if (deg < 90) {
						ry1 = y1 - 2
						ry2 = y2 - 2
					} else if (deg > 90) {
						ry1 = y1 + 2
						ry2 = y2 + 2
					} else {
						ry1 = y1
						ry2 = y2
					}
				}
			} else {
				rx1 = x1
				rx2 = x2
				ry1 = y1 + 2
				ry2 = y2 + 2
			}

			return {
				clipPath: `polygon(
					calc(${x1}px) calc(${y1}px),
					calc(${rx1}px) calc(${ry1}px),
					calc(${rx2}px) calc(${ry2}px),
					calc(${x2}px) calc(${y2}px)
				)`,
				backgroundColor: 'green'
			}
		},
	}
}
</script>

<style lang="scss">
	.chart {
		&__segment {
			position: absolute;
		}
	}
	.segment {
		width: 100%;
		height: 100%;
		display: inline-block;
	}
</style>