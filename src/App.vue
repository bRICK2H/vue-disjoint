<template>
	<div id="app">
		<div class="container">
			<div class="chart container__chart">
				<div class="chart__list chart__list">
					<span class="chart__item chart__item"
						v-for="(value, key) in coordsLine" :key="key"
						:style="draw(value)"
					>
					</span>
				</div>
			</div>
		</div>
	</div>
</template>

<script>

export default {
	name: 'App',
	components: {},
	data: () => ({
		json: {
			coords: {
				1: '100;377',
				2: '400;400',
				3: '400;100',
				4: '100;400'
			},
			// coords: {
			// 	1: '10;10',
			// 	2: '25;13',
			// 	3: '7;12',
			// 	4: '15;15'
			// }
		},
		lines: 3,
		points: 2,
		coordsLine: {},
		convertedCoords: [],
		res: 0,
	}),
	methods: {
		convertCoords() {
			return Object.entries(this.json.coords)
				.map(curr => {
					const [, value ] = curr;
					return value.split(';').map(s => +s)
				})
		},
		getMinMaxCoord(param = 'max') {
			this.convertedCoords = this.convertCoords()
			return param === 'max'
				? Math.max(...this.convertedCoords.flat())
				: Math.min(...this.convertedCoords.flat())
		},
		parseCoords() {
			return Object.entries(this.json.coords)
				.reduce((acc, curr) => {
					const [ dot, value ] = curr
					const [ x, y ] = value.split(';')

					acc[dot] = {
						x: (500 - 50) / this.getMinMaxCoord('max') * +x,
						y: (500 - 50) / this.getMinMaxCoord('max') * +y
					}

					return acc
				}, {})
		},
		getLines() {
			const coords = this.parseCoords()

			for (let i = 0; i < this.lines; i++) {
				this.coordsLine[i + 1] = []
				for (let j = 0; j < this.points; j++) {
					this.coordsLine[i + 1].push(coords[i + j + 1])
				}
			}
		},
		draw([{ x: x1, y: y1 }, { x: x2, y: y2 }]) {
			const deg = +(180 / Math.PI * Math.atan2(y2 - y1, x2 - x1)).toFixed(0)
			const sin = +Math.sin(deg * Math.PI / 180).toFixed(2)
			const isNegative = /-/.test(String(deg))
			let rx1, rx2, ry1, ry2 = 0
			console.log('DEG: ', +deg, sin)

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
				'clip-path': `polygon(
					calc(${x1}px) calc(${y1}px),
					calc(${rx1}px) calc(${ry1}px),
					calc(${rx2}px) calc(${ry2}px),
					calc(${x2}px) calc(${y2}px)
				)`,
			}
		}
	},
	created() {
		this.getLines()
		const res = (this.getMinMaxCoord('max') - this.getMinMaxCoord('min')) / 500
		this.res = res
	}
}
</script>

<style lang="scss">
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}
	.container {
		width: 100%;
		height: 100vh;
		display: flex;
		
		&__chart {
			margin: auto;
		}
	}
	.chart {
		width: 500px;
		height: 500px;
		border: 1px solid red;
		position: relative;

		&__list {
			width: 100%;
			height: 100%;
		}

		&__item {
			width: 100%;
			height: 100%;
			position: absolute;
			display: inline-block;
			background: green;
		}
	}
</style>
