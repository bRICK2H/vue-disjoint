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
				1: '100;100',
				2: '400;400',
				3: '350;100',
				4: '50;400'
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
		intersect() {
			console.log('intersect', this.coordsLine)
			const tmp = []
			const res = Object.entries(this.coordsLine).reduce((acc, curr) => {
				const [line, coords] = curr
				const [{ x: x1, y: y1 }, { x: x2, y: y2 }] = coords
				tmp.push(coords)
				console.log(line, x1, y1, x2, y2, tmp.slice(+line - 1))
				acc.push(line)
				return acc;
			}, [])
		},
		draw([{ x: x1, y: y1 }, { x: x2, y: y2 }]) {
			const deg = +(180 / Math.PI * Math.atan2(y2 - y1, x2 - x1)).toFixed(0)
			const sin = +Math.sin(deg * Math.PI / 180).toFixed(2)
			const isNegative = /-/.test(String(deg))
			// 1: '100;100', x1, y1
			// 2: '400;400', x2, y2
			// 3: '350;100', x3, y3
			// 4: '50;400'  x4, y4

			// x:=((x1 * y2 - x2 * y1) * (x4 - x3) - (x3 * y4 - x4 * y3) * (x2 - x1)) / ((y1 - y2) * (x4 - x3) - (y3 - y4) * (x2 - x1));
			const testX = ((200 * 400 - 400 * 377) * (50 - 350) - (350 * 400 - 50 * 100) * (400 - 200)) / ((377 - 400) * (50 - 350) - (100 - 400) * (400 - 200))
			// y:=((y3 - y4) * x - (x3 * y4 - x4 * y3)) / (x4 - x3);
			const testY = ((100 - 400) * testX - (350 * 400 - 50 * 100)) / (50 - 350)
			// (((x1 <= x) and (x2 >= x) and (x3 <= x) and (x4 >= x)) or ((y1 <= y) and (y2 >= y) and (y3 <= y) and(y4 >= y)))
			const cond = (((200 <= testX) && (400 >= testX) && (350 <= testX) && (50 >= testX)) || ((377 <= testY) && (400 >= testY) && (100 <= testY) && (400 >= testY)))
			// k1:= (x2 - x1) / (y2 - y1);
			const testK1 = (400 - 200) / (400 - 377)
			// k2:= (x4 - x3) / (y4 - y3);
			const testK2 = (50 - 350) / (400 - 100)
			console.log(testX, testY, cond, testK1 === testK2)
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

		this.intersect()
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
