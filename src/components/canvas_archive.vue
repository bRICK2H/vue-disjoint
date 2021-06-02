<template>
	<canvas class="canvas" :height="`${scale}`" :width="`${scale}`" ref='canvas'></canvas>
</template>

<script>
export default {
	name: 'cSegment',
	props: {
		points: {
			type: Array,
			default: () => ([])
		},
		index: {
			type: Number,
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
	data: () => ({}),
	methods: {
		scalable(point) {
			return (point * this.scale) / this.max
		}
	},
	watch: {
		points: {
			handler(points) {
				const ctx = this.$refs['canvas'].getContext('2d')
				const [{ x, y }] = this.points.splice(0, 1)
				ctx.lineWidth = 2
				ctx.strokeStyle = 'green'

				ctx.beginPath()
				ctx.clearRect(0, 0, this.scale, this.scale)
				ctx.moveTo(this.scalable(x), this.scalable(y))

				this.points.forEach(curr => {
					const { x, y } = curr
					ctx.lineTo(this.scalable(x), this.scalable(y))
				});

				ctx.stroke()
			}
		}
	}
}
</script>

<style lang="scss">
	.canvas {
		transition: .3s;
	}
</style>