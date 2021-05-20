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
				2: '400;400',
				3: '400;100',
				4: '100;400'
			}
			// coords: {
			// 	1: '10;10',
			// 	2: '25;13',
			// 	3: '7;12',
			// 	4: '15;15'
			// }
		},
		lines: 3,
		points: 2,
		coordsLine: {}
	}),
	methods: {
		parseCoords() {
			return Object.entries(this.json.coords)
				.reduce((acc, curr) => {
					const [ dot, value ] = curr
					const [ x, y ] = value.split(';')

					acc[dot] = { x: +x, y: +y }

					return acc
				}, {})
		},
		getLines() {
			const coords = this.parseCoords()
			// console.log('coor: ', Object.entries(coords).map(c => Object.values(c[1])))

			for (let i = 0; i < this.lines; i++) {
				this.coordsLine[i + 1] = []
				for (let j = 0; j < this.points; j++) {
					this.coordsLine[i + 1].push(coords[i + j + 1])
				}
			}
		},
		drow([f, s]) {
			return {
				'clip-path': `polygon(
					calc(${f.x}px - 1px) calc(${f.y}px + 1px),
					calc(${f.x}px + 1px) calc(${f.y}px + 1px),
					calc(${s.x}px + 1px) calc(${s.y}px + 1px),
					calc(${s.x}px - 1px) calc(${s.y}px + 1px)
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
