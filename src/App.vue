<template>
	<div id="app">
		<div class="container">
			<button @click="createDisjoint">Изменить отрезки</button>
			<div class="chart container__chart">
				<c-segment v-for="(segment, key) in segments"
					:key="`segment${key}`"
					:segment="segment"
				/>
				<c-point v-for="(point, key) in points"
					:key="`point${key}`"
					:primaryCoords="primaryCoords"
					:point="point"
					:keyPoint="key"
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
				1: '12;10',
				2: '25;13',
				3: '90;12',
				4: '13;19',
				5: '30;5',
				6: '28;17',
				7: '26;9',
				8: '74;84',
				9: '68;18',
				10: '30;38',
				11: '15;50',
				12: '80;33',
				13: '99;6',
				14: '18;30',
				15: '50;55',
				16: '84;46',
				17: '18;25',
				18: '66;90',
				19: '5;40',
				20: '27;43'
			},
		}),
		computed: {
			convertCoords() {
				return Object.entries(this.primaryCoords)
					.map(curr => {
						const [point, value] = curr;

						return { [point]: value.split(';').map(pt => +pt) }
					})
			},
			getMaxCoord() {
				const deployedCoords = this.convertCoords.map(curr => Object.values(curr).flat()).flat()
				return Math.max(...deployedCoords)
			},
			scalableCoords() {
				return this.convertCoords.map(curr => {
					const [point, value] = Object.entries(curr).flat()

					return this.convertCoords.length > 1
						? { [point]: value.map(c => c * 500 / this.getMaxCoord) }
						: { [point]: value.map(c => c) }
				})
			},
			points() {
				return this.scalableCoords.reduce((acc, curr, i) => {
					const [point, [x, y]] = Object.entries(curr).flat()
					acc[point] = { x, y }

					return acc
				}, {})
			},
			segments() {
				return this.getSegments(this.points)
			},
		},
		methods: {
			getSegments(points) {
				const res = {}

				for (let i = 0; i < Object.keys(points).length; i++) {
					const [point] = Object.entries(points)[i]
					const [[, points1], p2] = Object.entries(points).slice(i, i + 2)

					res[point] = [points1, p2 === undefined ? points1 : p2[1]]
				}

				return res
			},
			checkIntersecting(segments) {
				const restValues = point => Object.entries(segments).filter(curr => +curr[0] !== +point)

				return Object.entries(segments).every(curr => {
					const [point, segment] = curr
					const [{ x: x1, y: y1 }, { x: x2, y: y2 }] = segment

					return restValues(+point).some(c => {
						const [, [{ x: x3, y: y3 }, { x: x4, y: y4 }]] = c,
							v1 = (x4 - x3) * (y1 - y3) - (y4 - y3) * (x1 - x3),
							v2 = (x4 - x3) * (y2 - y3) - (y4 - y3) * (x2 - x3),
							v3 = (x2 - x1) * (y3 - y1) - (y2 - y1) * (x3 - x1),
							v4 = (x2 - x1) * (y4 - y1) - (y2 - y1) * (x4 - x1)

						return (v1 * v2 < 0) && (v3 * v4 < 0);
					})
				})
			},
			createDisjoint() {
				const disjointPoints = Object
					.entries(this.convertCoords)
					.map((curr, i) => {
						const [, { [i + 1]: points }] = curr

						return points
					})
					.sort(([x1, y1], [x2, y2]) => x1 - x2 || y1 - y2)

				const formatDisjointTotalResult = disjointPoints.reduce((acc, curr, i) => {
					const [x, y] = curr
					acc[i + 1] = `${x};${y}`

					return acc
				}, {})

				const formatDisjointPoints = disjointPoints.reduce((acc, curr, i) => {
					const [x, y] = curr
					acc[i + 1] = { x, y }

					return acc
				}, {})

				return !this.checkIntersecting(this.getSegments(formatDisjointPoints))
					? this.primaryCoords = formatDisjointTotalResult
					: this.createDisjoint()
			},
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

	html,
	body {
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
		align-items: center;

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