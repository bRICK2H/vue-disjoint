<template>
	<div id="app">
		<div class="container">
			<div class="chart container__chart">
				<div class="chart__list chart__list">
					<span class="chart__item chart__item"
						v-for="(value, key) in coordsLine" :key="key"
						:style="drow(value)"
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
				3: '400;100',
				2: '400;400',
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
		convertedCoords: []
	}),
	methods: {
		convertCoords() {
			return Object.entries(this.json.coords)
				.map(curr => {
					const [, value ] = curr;
					return value.split(';').map(s => +s)
				})
		},
		// getMaxCoord() {
		// 	this.convertedCoords = this.convertCoords()
		// 	console.log(this.convertedCoords)
		// 	return Math.max(...this.convertedCoords.flat())
		// },
		parseCoords() {
			return Object.entries(this.json.coords)
				.reduce((acc, curr) => {
					const [ dot, value ] = curr
					const [ x, y ] = value.split(';')

					// acc[dot] = {
					// 	x: 500 / this.getMaxCoord() * +x,
					// 	y: 500 / this.getMaxCoord() * +y
					// }

					acc[dot] = {
						x: +x,
						y: +y
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

			console.log(this.coordsLine)
		},
		drow([{ x: x1, y: y1 }, { x: x2, y: y2 }]) {
			console.log('v', x1, y1, x2, y2)
			// const deg = y2 - y1 / x2 - x1
			// console.log(Math.atan(deg))

			// return {
			// 			'clip-path': `polygon(
			// 				calc(${f.x - 1}px) calc(${f.y >= f.x ? f.y + 1 : f.y - 1 }px),
			// 				calc(${f.x + 1}px) calc(${f.y < f.x ? f.y + 1 : f.y - 1 }px),
			// 				calc(${s.x + 1}px) calc(${s.y <= f.x ? s.y + 1 : s.y - 1 }px),
			// 				calc(${s.x - 1}px) calc(${s.y > f.x ? s.y + 1 : s.y - 1 }px)
			// 			)`,
			// 		}
			
			// const deg = 
			return {
						'clip-path': `polygon(
							calc(${x1 - 2}px) calc(${y1 + 2}px),
							calc(${x1 + 2}px) calc(${y1 - 2}px),
							calc(${x2 + 2}px) calc(${y2 - 2}px),
							calc(${x2 - 2}px) calc(${y2 + 2}px)
						)`,
					}
		}
	},
	created() {
		this.getLines()
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
		width: 550px;
		height: 550px;
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
