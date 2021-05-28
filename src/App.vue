<template>
	<div id="app">
		<div class="container">
			<div class="chart container__chart">
				<c-segment
					v-for="(segments, key) in getSegments" :key="`cl${key}`"
					:segments="segments"
				/>
				<c-point
					v-for="(points, key, i) in formatScalableCoords" :key="`scp${key}`"
					@fillingCoords="fillingCoords(points, i)"
					:filling="filling"
					:isEnd="isEnd"
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
		// primaryCoords: {
		// 	1: '100;100',
		// 	2: '400;400',
		// 	3: '400;100',
		// 	4: '100;400',
		// 	5: '300;50',
		// },
		// primaryCoords: {
		// 	1: '150;450',
		// 	2: '900;400',
		// 	3: '400;500',
		// 	4: '200;400',
		// },
		primaryCoords: {
			1: '10;10',
			2: '25;13',
			3: '7;12',
			4: '15;15',
			5: '15;5',
			6: '8;15'
		},
		// primaryCoords: {
		// 	1: '17;10', // +
		// 	2: '25;1', // +
		// 	3: '7;12',
		// 	4: '15;15', // + 
		// 	5: '15;5', // +
		// 	6: '8;15' // +
		// },
		clonePrimaryCoords: {},
		filling: [],
		points: 2,
		isEnd: false,
	}),
	computed: {
		convertCoords() {
			return Object.entries(this.primaryCoords)
				.map(curr => {
					const [, value ] = curr;
					return value.split(';').map(pt => +pt)
				})
		},
		getMaxCoord() {
			return Math.max(...this.convertCoords.flat())
		},
		scalableCoords() {
			return this.convertCoords.map(curr => {
				return this.convertCoords.length > 1
					? curr.map(c => c * 500 / this.getMaxCoord)
					: curr.map(c => c)
			})
		},
		formatScalableCoords() {
			return this.scalableCoords.reduce((acc, curr, i) => {
				const [ x, y ] = curr
				acc[i + 1] = { x, y }

				return acc
				}, {})
		},
		getSegments() {
			const result = {}
			const initLength = Object.keys(this.formatScalableCoords).length
			const amountSegments = initLength > 1 ? initLength - 1 : initLength

			for (let i = 0; i < amountSegments; i++) {
				result[i + 1] = []
				for (let j = 0; j < this.points; j++) {
					result[i + 1].push({
						...this.formatScalableCoords[i + j + 1],
						point: i + j + 1
					})
				}
			}
			
			return result
		},
		intersect() {
			const restValues = oLine => Object.entries(this.getSegments).filter(curr => +curr[0] !== +oLine)

			return Object.entries(this.getSegments).reduce((acc, curr) => {
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
					result: isIntersect,
					points: segments.map(seg => seg.point) 
				})

				return acc;
			}, [])
		},
	},
	methods: {
		fillingCoords(point, i) {
			console.log(i)
			this.filling.push(JSON.stringify(point))
		},
		changeSegments() {
			const isIntersect = !this.intersect.every(curr => !curr.result)
			if (!isIntersect) return
			
			let doublePoints = this.intersect
				.filter(curr => curr.result)
				.map(curr => curr.points)
				.flat()
				.reduce((acc, curr, i, arr) => {
					const isDouble = arr.some((el, ii) => i !== ii && curr === el)
					if (isDouble) {
						acc.push(curr)
					}

					return acc
				}, [])
			
			const unique = Array.from(new Set(doublePoints))
			console.log({unique})


			if (unique.length > 1) {
				console.log('no unique', this.intersect, unique)

				// setTimeout(() => {
					unique.forEach((el) => {
						this.primaryCoords[el] = this.primaryCoords[el + 1]
					})

					this.changeSegments()
					setTimeout(() => {
						console.log('done1')
					})
				// }, 500)
			} else {
				console.log('is unique', unique)
				const res = this.intersect
					.filter(curr => !curr.result)
					.reduce((acc, curr) => {
						const { points } = curr
						const [p1, p2] = points
						const r1 = this.primaryCoords[p2]
						const r2 = this.primaryCoords[p1]
	
						// setTimeout(() => {
							this.primaryCoords[p1] = r1
							this.primaryCoords[p2] = r2
							this.primaryCoords[Object.keys(this.primaryCoords).length] = this.primaryCoords[1]
						// }, 1000)
						
	
						acc.push(points)
						return acc
				}, [])
	
				console.log(res)
				setTimeout(() => {
					console.log('done2')
				})
			}

			setTimeout(() => {
				this.isEnd = true
				console.log('done3')
			}, 1000)
			
		}
	},
	created() {
		this.clonePrimaryCoords = Object.assign({}, this.primaryCoords)
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
