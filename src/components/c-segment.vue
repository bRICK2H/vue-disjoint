<template>
	<span class="segment chart__segment"
		:style="draw"
	></span>
</template>

<script>
export default {
	name: 'cSegment',
	props: {
		segment: {
			type: Array,
			default: () => ([])
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
	data: () => ({}),
	computed: {
		draw() {
			let thickness = 0.0015 * this.max
			let rx1 = 0, rx2 = 0, ry1 = 0, ry2 = 0
			const [{ x: x1, y: y1 }, { x: x2, y: y2 }] = this.segment,
					deg = +(180 / Math.PI * Math.atan2(y2 - y1, x2 - x1)).toFixed(0)

			if (deg) {
				rx1 = x1 - thickness
				rx2 = x2 - thickness

				if (/-/.test(String(deg))) {
					if (deg < -90) {
						ry1 = y1 + thickness
						ry2 = y2 + thickness
					} else if (deg > -90) {
						ry1 = y1 - thickness
						ry2 = y2 - thickness
					} else {
						ry1 = y1
						ry2 = y2
					}
				} else {
					if (deg < 90) {
						ry1 = y1 + thickness
						ry2 = y2 + thickness
					} else if (deg > 90) {
						ry1 = y1 - thickness
						ry2 = y2 - thickness
					} else {
						ry1 = y1
						ry2 = y2
					}
				}
			} else {
				rx1 = x1
				rx2 = x2
				ry1 = y1 + thickness
				ry2 = y2 + thickness
			}

			return {
				clipPath: `polygon(
					calc(${this.scalable(x1)}px) calc(${this.scalable(y1)}px),
					calc(${this.scalable(rx1)}px) calc(${this.scalable(ry1)}px),
					calc(${this.scalable(rx2)}px) calc(${this.scalable(ry2)}px),
					calc(${this.scalable(x2)}px) calc(${this.scalable(y2)}px)
				)`,
				backgroundColor: 'green'
			}
		},
	},
	methods: {
		scalable(point) {
			return (point * this.scale) / this.max
		}
	},
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
		transition: .5s;
	}
</style>