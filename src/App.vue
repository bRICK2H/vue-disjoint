<template>
	<div id="app">
		<div class="container">
			
			<!-- Interface -->
			<cInterface
				:points="coords"
				@create-disjoint="createDisjoint"
			/>

			<!-- Chart -->
			<div class="chart container__chart"
				:style="styleChart"
			>
				<c-segment v-for="(segment, i) in segments"
					:key="`segment${i}`"
					:max="maxPoint"
					:scale="scale"
					:segment="segment"
				/>
				<c-point v-for="(point, i) in points"
					:key="`point${i}`"
					:max="maxPoint"
					:scale="scale"
					:point="point"
					:keyPoint="point.keyPoint"
				/>
			</div>

		</div>
	</div>
</template>

<script>
	import cPoint from '@/components/c-point'
	import cSegment from '@/components/c-segment'
	import cInterface from '@/components/c-interface'

	export default {
		name: 'App',
		components: {
			cPoint,
			cSegment,
			cInterface
		},
		data: () => ({
			coords: [],
			scale: 0
		}),
		computed: {
			styleChart() {
				return {
					width: `${this.scale * 0.1}rem`,
					height: `${this.scale * 0.1}rem`,
				}
			},
			maxPoint() {
				const deployedCoords = this.coords.map(curr => Object.values(curr).flat()).flat()
				return Math.max(...deployedCoords)
			},
			points() {
				return this.coords.reduce((acc, curr) => {
					const [point, [x, y]] = Object.entries(curr).flat()
					acc.push({ x, y, keyPoint: point })

					return acc
				}, [])
			},
			segments() {
				if (this.coords.length < 2) return

				const amountSegments = this.coords.length > 2 ? this.coords.length - 1 : 1
				const collectionSegments = []
				for (let i = 0; i < amountSegments; i++) {
					const segments = this.coords.slice(i, i + 2)
					const m_segments = segments.map(curr => {
						const [x, y] = Object.values(curr).flat()
						return { x, y }
					})
					const [p1, p2] = m_segments

					collectionSegments.push(p2 === undefined ? [p1, p1] : m_segments)
				}

				return collectionSegments
			},
		},
		methods: {
			createDisjoint(points) {
				this.coords = Object
					.entries(points)
					.map(curr => {
						const [, o] = curr
						return { point: +Object.keys(o)[0], points: Object.values(o).flat() }
					})
					.sort(({ points: [x1, y1] }, { points: [x2, y2] }) => {
						return x1 - x2 || y1 - y2
					})
					.reduce((acc, curr) => {
						const { point, points } = curr
						acc.push({ [point]: points })

						return acc
					}, [])
			},
			convertPoints(primary) {
				return Object.entries(primary)
					.map(curr => {
						const [point, value] = curr;

						return { [point]: value.split(';').map(pt => +pt) }
					})
			},
		},
		created() {
			setTimeout(() => {
				this.coords = this.convertPoints(require('./api/data.json'))
			}, 500)
		},
		mounted() {
			this.scale = document.body.offsetHeight - 200
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

	html,
	body {
		font-size: 62.5%;
		font-family: 'Ubuntu', sans-serif;
	}

	body {
		width: 100%;
		font-size: 1.6rem;
		font-weight: 100;
	}

	.container {
		width: 100%;
		height: 100vh;
		display: flex;

		&__chart {
			margin: auto 0;
		}
		&__interface {
			margin: auto 10rem;
		}
	}

	.chart {
		position: relative;

		&::before, &::after {
			content: '';
			position: absolute;
			bottom: 0;
			left: 0;
			background: #999;
		}
		&::before {
			height: 100%;
			width: 1px;
		} 
		&::after {
			width: 100%;
			height: 1px;
		} 
	}
</style>