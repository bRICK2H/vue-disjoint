<template>
	<div id="app">
		<div class="container">
			<div class="chart container__chart">
				<c-segment
					v-for="(segments, key) in coordLines" :key="`cl${key}`"
					:segments="segments"
				/>
				<c-point
					v-for="(points, key) in formatedScalableCoords" :key="`scp${key}`"
					:primaryCoords="primaryCoords"
					:points="points"
					:nPoint="key"
				/>
			</div>
		</div>
	</div>
</template>

<script>
import cPoint from '@/components/c-point'
import cSegment from '@/components/c-segment'

export default {
	name: 'App',
	components: {
		cSegment,
		cPoint
	},
	data: () => ({
		primaryCoords: {
			1: '100;100',
			2: '400;400',
			3: '400;100',
			4: '100;400',
		},
		// primaryCoords: {
		// 	1: '10;10',
		// 	2: '25;13',
		// 	3: '7;12',
		// 	4: '15;15'
		// },
		points: 2,
		coordLines: {},
		convertedCoords: [],
		formatedScalableCoords: [],
		res: 0,
	}),
	methods: {
		convertCoords() {
			return Object.entries(this.primaryCoords)
				.map(curr => {
					const [, value ] = curr;
					return value.split(';').map(s => +s)
				})
		},
		getMaxCoord() {
			return Math.max(...this.convertedCoords.flat())
		},
		scalableCoords() {
			return this.convertedCoords.map(curr => {
				return this.convertedCoords.length > 1
					? curr.map(c => c * 500 / this.getMaxCoord())
					: curr.map(c => c)
			})
		},
		formatScalableCoords() {
			const scalableCoords = this.scalableCoords()

			return scalableCoords.reduce((acc, curr, i) => {
				const [ x, y ] = curr
				acc[i + 1] = { x, y }

				return acc
				}, {})
		},
		getSegments() {
			const initLength = Object.keys(this.formatedScalableCoords).length
			const amountSegments = initLength > 1 ? initLength - 1 : initLength

			for (let i = 0; i < amountSegments; i++) {
				this.coordLines[i + 1] = []
				for (let j = 0; j < this.points; j++) {
					this.coordLines[i + 1].push({
						...this.formatedScalableCoords[i + j + 1],
						point: i + j + 1
					})
				}
			}
		},
		intersect() {
			const restValues = oLine => Object.entries(this.coordLines).filter(curr => +curr[0] !== +oLine)

			return Object.entries(this.coordLines).reduce((acc, curr) => {
				const [line, segments] = curr
				const [{ x: x1, y: y1 }, { x: x2, y: y2 }] = segments
				const isIntersect = restValues(+line).some(c => {
					const [, [{ x: x3, y: y3 }, { x: x4, y: y4 }]] = c,
						v1 = (x4 - x3) * (y1 - y3) - (y4 - y3) * (x1 - x3),
						v2 = (x4 - x3) * (y2 - y3) - (y4 - y3) * (x2 - x3),
						v3 = (x2 - x1) * (y3 - y1) - (y2 - y1) * (x3 - x1),
						v4 = (x2 - x1) * (y4 - y1) - (y2 - y1) * (x4 - x1)

					return (v1 * v2 < 0) && (v3 * v4 < 0);
				})

				acc.push({
					res: isIntersect,
					nSegment: +line,
					segments
				})

				return acc;
			}, [])
		},
		changeSegments() {
			const resultIntersect = this.intersect()
			const isIntersect = !resultIntersect.every(curr => !curr.res)

			if (!isIntersect) return
			
			console.log({resultIntersect}, isIntersect)
			const res = resultIntersect.reduce((acc, curr) => {
				const { res, nSegment, segments } = curr
				if (res) {
					const normalSegment = resultIntersect.find(s => !s.res)
					console.log({normalSegment})
				}

				acc = 1
				return acc
			})
		}
	},
	created() {
		this.convertedCoords = this.convertCoords()
		this.formatedScalableCoords = this.formatScalableCoords()
		this.getSegments()
		this.changeSegments()
	}
}
</script>

<style lang="scss">
	@import url('https://fonts.googleapis.com/css2?family=Ubuntu:wght@300;400;500;700&display=swap');

	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}
	html {
		font-size: 62.5%;
	}
	html, body {
		font-family: 'Ubuntu', sans-serif;
	}
	body {
		font-size: 1.6rem;
		font-weight: 100;
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
		width: 650px;
		height: 650px;
		border-left: 1px solid #000;
		border-bottom: 1px solid #000;
		position: relative;
	}
</style>
