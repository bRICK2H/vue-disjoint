<template>
	<div id="app">
		<div class="container">
			<button
				@click="test4"
			>Изменить отрезки</button>
			<div class="chart container__chart">
				<c-segment
					v-for="(segments, key) in getSegments" :key="`cl${key}`"
					:segments="segments"
				/>
				<c-point
					v-for="(points, key) in formatScalableCoords" :key="`scp${key}`"
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
		// 	2: '600;500',
		// 	3: '600;0',
		// 	4: '100;600',
		// 	5: '150;50',
		// 	6: '90;150'
		// },
		// primaryCoords: {
		// 	1: '10;10',
		// 	2: '200;200',
		// 	3: '200;100',
		// 	4: '180;250',
		// 	5: '180;99',
		// 	6: '170;220',
		// 	7: '150;10',
		// 	8: '400;15',
		// 	9: '15;250'
		// },
		// primaryCoords: {
		// 	1: '10;10',
		// 	2: '25;13',
		// 	3: '7;12',
		// 	4: '15;15',
		// 	5: '30;10',
		// 	6: '28;15',
		// 	7: '29;12',
		// 	8: '15;9',
		// 	9: '6;18',
		// 	10: '10;18'
		// },
		primaryCoords: {
			1: '100;100',
			2: '400;400',
			3: '400;100',
			4: '100;400',
			5: '300;50',
		},
		// primaryCoords: {
		// 	1: '150;450',
		// 	2: '900;400',
		// 	3: '400;500',
		// 	4: '200;400',
		// },
		// primaryCoords: { // -
		// 	1: '10;10',
		// 	2: '25;13',
		// 	3: '7;12',
		// 	4: '15;15',
		// 	5: '15;5',
		// 	6: '8;15'
		// },
		// primaryCoords: {
		// 	1: '17;10', // +
		// 	2: '25;1', // +
		// 	3: '7;12',
		// 	4: '15;15', // + 
		// 	5: '15;5', // +
		// 	6: '8;15' // +
		// },
		points: 2,
		counter: 0,
		temp: '',
		temp2: '',
		prevDj: '',
		prevUnique: '',
	}),
	computed: {
		convertCoords() {
			// console.log('primCoo', this.primaryCoords)
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
			const points = Object.keys(this.formatScalableCoords).length === 1 ? 1 : this.points

			for (let i = 0; i < amountSegments; i++) {
				result[i + 1] = []
				for (let j = 0; j < points; j++) {
					result[i + 1].push({ ...this.formatScalableCoords[i + j + 1], point: i + j + 1 })
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

		test4() {
			if (Object.keys(this.primaryCoords).length <= 1) return

			const getDisjoint = () => this.intersect.filter(f => !f.result).map(m => m.points)
			const getInterseptions = () => this.intersect.filter(curr => curr.result).map(curr => curr.points)

			getDisjoint().forEach(arr => {
				const [p1, p2] = arr

				if (getInterseptions().flat().includes(p1) && getInterseptions().flat().includes(p2)) {
					console.log('RES', arr)
					const [p1, p2] = arr,
					cp1 = this.primaryCoords[p1],
					cp2 = this.primaryCoords[p2]
	
	
					// console.log({p1, p2})
					this.primaryCoords[p1] = cp2
					this.primaryCoords[p2] = cp1
				}
			})

			console.log(getInterseptions(), getDisjoint())

			const res = getDisjoint()
				.flat()
				.reduce((acc, curr, i, arr) => {
					const isDouble = arr.some((el, ii) => i !== ii && curr === el)
					if (isDouble) acc.push(curr)

					return acc
				}, [])
			
			const uniqueDisjoint = Array.from(new Set(res))

			console.log({uniqueDisjoint})
			
			if (uniqueDisjoint.length && this.prevUnique !== JSON.stringify(uniqueDisjoint)) {
				const uniqueSegment = getDisjoint().reduce((acc, curr) => {
					if (curr.includes(...uniqueDisjoint)) {
	
						if (!acc[uniqueDisjoint]) {
							acc[uniqueDisjoint] = []
						}
	
						acc[uniqueDisjoint].push(...curr.filter(c => !uniqueDisjoint.includes(c)))
					}
	
					return acc
				}, {})

				console.log({uniqueSegment})
				Object.entries(uniqueSegment).forEach(curr => {
					const [,[p1, p2]] = curr,
					cp1 = this.primaryCoords[p1],
					cp2 = this.primaryCoords[p2]

					this.primaryCoords[p1] = cp2
					this.primaryCoords[p2] = cp1
				})

				this.prevUnique = JSON.stringify(uniqueDisjoint)
			} else {
				console.log('no uniq')
				const isAllintersections = this.intersect.every(curr => curr.result)
				console.log(isAllintersections)
				// const [[p1, p2]] = getInterseptions().splice(0, 2),
				// 	cp1 = this.primaryCoords[p1],
				// 	cp2 = this.primaryCoords[p2]


				// this.primaryCoords[p1] = cp2
				// this.primaryCoords[p2] = cp1
				if (isAllintersections) {
					const defineIntersections = this.intersect.filter(curr => curr.result).map(curr => curr.points)
					const allowedIntersect = () => {
						console.log('all inter')
						const [[p1, p2]] = defineIntersections.splice(0, 2),
							cp1 = this.primaryCoords[p1],
							cp2 = this.primaryCoords[p2]


						this.primaryCoords[p1] = cp2
						this.primaryCoords[p2] = cp1
					}

				
					allowedIntersect()
					while(isAllintersections && defineIntersections.length) {
						console.log('::', isAllintersections,  defineIntersections.length)
						allowedIntersect()
					}
				} else {
					getInterseptions().forEach(curr => {
						const [p1, p2] = curr,
								cp1 = this.primaryCoords[p1],
								cp2 = this.primaryCoords[p2]
	
	
						this.primaryCoords[p1] = cp2
						this.primaryCoords[p2] = cp1
					})
				}

			}

			
			
			const isDisjoint = this.intersect.every(curr => !curr.result)
			console.log({isDisjoint}, this.intersect)
			// return !isDisjoint ? this.test3() : false
		},

		test3() {
			if (Object.keys(this.primaryCoords).length <= 1) return


			const disjoint = this.intersect
				.filter(f => !f.result)
				.map(m => m.points)
			console.log({disjoint}, this.intersect)

			const intersections = this.intersect.every(curr => curr.result)
			const inter = this.intersect.filter(curr => curr.result).map(curr => curr.points)
			
			console.log('INTER', inter, disjoint, intersections)
			if(intersections) {
				const allowedIntersect = () => {
					console.log('all inter')
					const [[p1, p2]] = inter.splice(0, 2),
						cp1 = this.primaryCoords[p1],
						cp2 = this.primaryCoords[p2]


					this.primaryCoords[p1] = cp2
					this.primaryCoords[p2] = cp1
				}

				console.log('::', intersections, inter)
				while(intersections && inter.length) {
					allowedIntersect()
				}
				// return intersections ? allowedIntersect() : false
			} else if (JSON.stringify(disjoint) === this.prevDj) {
				console.log('pizda vstali sbrasivaem vse', JSON.stringify(disjoint), this.prevDj)
				this.test(3)
				// const [[p1, p2]] = disjoint.splice(0, 2),
				// 		cp1 = this.primaryCoords[p1],
				// 		cp2 = this.primaryCoords[p2]


				// this.primaryCoords[p1] = cp2
				// this.primaryCoords[p2] = cp1
			}

			this.prevDj = JSON.stringify(disjoint)

			
			console.log('before исключение', inter.length)
			if(inter.length) {
				console.log('исключение')
				if (JSON.stringify(disjoint) !== this.prevDj) {
					const dj = this.intersect.filter(f => !f.result).map(m => m.points)
					console.log('исключение if')
					dj.forEach(arr => {
						const [p1, p2] = arr
						console.log({p1, p2})
						if (inter.flat().includes(p1) && inter.flat().includes(p2)) {
							console.log('RES', arr)
							const [p1, p2] = arr,
							cp1 = this.primaryCoords[p1],
							cp2 = this.primaryCoords[p2]
			
			
							// console.log({p1, p2})
							this.primaryCoords[p1] = cp2
							this.primaryCoords[p2] = cp1
						}
					})

					
				} else {
					disjoint.forEach(curr => {
						const [p1, p2] = curr,
								cp1 = this.primaryCoords[p1],
								cp2 = this.primaryCoords[p2]

						this.primaryCoords[p1] = cp2
						this.primaryCoords[p2] = cp1
						console.log('if disjoint', disjoint, this.intersect.filter(curr => curr.result).map(curr => curr.points))
					})

					const res = disjoint.filter(arr => {
						const [p1, p2] = arr
						console.log({p1, p2})
						return (inter.flat().includes(p1) && inter.flat().includes(p2))
					})
					
					if (res.length === 1) {
						console.log('исключение ELSE', res)
						const [[p1, p2]] = res,
							cp1 = this.primaryCoords[p1],
							cp2 = this.primaryCoords[p2]
			
			
							// console.log({p1, p2})
							this.primaryCoords[p1] = cp2
							this.primaryCoords[p2] = cp1
						this.test3()
					}
				}
			}

			const isDisjoint = this.intersect.every(curr => !curr.result)
			console.log({isDisjoint}, this.intersect)
			// return !isDisjoint ? this.test3() : false
		},

		test2() {
			// Если определены 2 отрезка, завершаем работу
			if (Object.keys(this.primaryCoords).length <= 1) return


			const disjoint = this.intersect
				.filter(f => !f.result)
				.map(m => m.points)
			const intersecting = this.intersect
				.filter(c => c.result)
				.map(c => c.points)
				.flat()
				.reduce((acc, curr, i, arr) => {
					const isDouble = arr.some((el, ii) => i !== ii && curr === el)
					if (isDouble) acc.push(curr)

					return acc
				}, [])
			const uniqueIntersections = Array.from(new Set(intersecting))
			
				
			if (this.temp !== JSON.stringify(disjoint)) {
				this.temp = JSON.stringify(disjoint)
				disjoint.forEach(curr => {
					const [p1, p2] = curr,
							cp1 = this.primaryCoords[p1],
							cp2 = this.primaryCoords[p2]
	
	
					// console.log({p1, p2})
					this.primaryCoords[p1] = cp2
					this.primaryCoords[p2] = cp1
					console.log('if disjoint', disjoint, this.intersect.filter(curr => curr.result).map(curr => curr.points))
				})

				const inter = this.intersect.filter(curr => curr.result).map(curr => curr.points)
				const dj = this.intersect.filter(f => !f.result).map(m => m.points)
				console.log('INTER', inter, dj)
				if(inter.length) {
					dj.forEach(arr => {
						const [p1, p2] = arr
						console.log({p1, p2})
						if (inter.flat().includes(p1) && inter.flat().includes(p2)) {
							console.log('RES', arr)
							const [p1, p2] = arr,
							cp1 = this.primaryCoords[p1],
							cp2 = this.primaryCoords[p2]
			
			
							// console.log({p1, p2})
							this.primaryCoords[p1] = cp2
							this.primaryCoords[p2] = cp1
						}
					})
					
				}
			} else {
				console.log('else')
				const inter = this.intersect.filter(curr => curr.result).map(curr => curr.points)
				
				if (inter.length) {
					console.log('inter', inter)
					const res = inter.filter((arr, i) => {

						return disjoint.some(curr => {
							const [p1, p2] = curr
							return arr.includes(p1) || arr.includes(p2)
						})
						
					})
					

					console.log(this.temp2 === JSON.stringify(res), this.temp, disjoint)
					this.temp2 = JSON.stringify(res)
					if (res.length) {
						res.forEach(curr => {
							const [p1, p2] = curr,
							cp1 = this.primaryCoords[p1],
							cp2 = this.primaryCoords[p2]
		
							this.primaryCoords[p1] = cp2
							this.primaryCoords[p2] = cp1
						})
					} else {
						console.log(uniqueIntersections)
						// uniqueIntersections.forEach(curr => {
						// 	const [p1, p2] = uniqueIntersections.splice(0, 2),
						// 	cp1 = this.primaryCoords[p1],
						// 	cp2 = this.primaryCoords[p2]
		
						// 	this.primaryCoords[p1] = cp2
						// 	this.primaryCoords[p2] = cp1

						// 	if (uniqueIntersections.length === 1) {
						// 		console.log('here', res, uniqueIntersections)
						// 		const matchRest = inter.filter(curr => curr.includes(...uniqueIntersections))

						// 		matchRest.forEach(curr => {
						// 			const [p1, p2] = curr,
						// 			cp1 = this.primaryCoords[p1],
						// 			cp2 = this.primaryCoords[p2]
				
						// 			this.primaryCoords[p1] = cp2
						// 			this.primaryCoords[p2] = cp1
						// 		})
						// 	}
						// })
					}
				}

			}

				
			// console.log(uniqueIntersections)

			// }
			
			const isDisjoint = this.intersect.every(curr => !curr.result)
			console.log({isDisjoint}, this.intersect)
			// return !isDisjoint ? this.test2() : false
			
		},
		
		test() {
			// Если определены 2 отрезка, завершаем работу
			if (Object.keys(this.primaryCoords).length <= 1) return


			const disjoint = this.intersect
				.filter(f => !f.result)
				.map(m => m.points)
			const intersecting = this.intersect
				.filter(c => c.result)
				.map(c => c.points)
				.flat()
				.reduce((acc, curr, i, arr) => {
					const isDouble = arr.some((el, ii) => i !== ii && curr === el)
					if (isDouble) acc.push(curr)

					return acc
				}, [])
			const uniqueIntersections = Array.from(new Set(intersecting))


			// console.log('start', 'disjoint', disjoint, intersecting, this.intersect)
			
				// const next = () => {
				// 	const [[p1, p2]] = disjoint.splice(0, 2),
				// 	cp1 = this.primaryCoords[p1],
				// 	cp2 = this.primaryCoords[p2]

				// 	this.primaryCoords[p1] = cp2
				// 	this.primaryCoords[p2] = cp1
				// }

				// setTimeout(() => {
				// 	if (this.intersect.some(curr => curr.result)) {
				// 		console.log('next')
				// 		next()
				// 	}
				// }, 1000)
				
			disjoint.forEach(curr => {
				const [p1, p2] = curr,
						cp1 = this.primaryCoords[p1],
						cp2 = this.primaryCoords[p2]

				console.log({p1, p2})
				this.primaryCoords[p1] = cp2
				this.primaryCoords[p2] = cp1
			})

			const inter = this.intersect.filter(curr => curr.result).map(curr => curr.points)
			
			if (inter.length) {
				console.log('inter', inter)
				const res = inter.filter((arr, i) => {

					return disjoint.some(curr => {
						const [p1, p2] = curr
						return arr.includes(p1) || arr.includes(p2)
					})
					
				})
				
				console.log({res})
				if (res.length) {
					res.forEach(curr => {
						const [p1, p2] = curr,
						cp1 = this.primaryCoords[p1],
						cp2 = this.primaryCoords[p2]
	
						this.primaryCoords[p1] = cp2
						this.primaryCoords[p2] = cp1
					})
				} else {
					console.log(uniqueIntersections)
					uniqueIntersections.forEach(curr => {
						const [p1, p2] = uniqueIntersections.splice(0, 2),
						cp1 = this.primaryCoords[p1],
						cp2 = this.primaryCoords[p2]
	
						this.primaryCoords[p1] = cp2
						this.primaryCoords[p2] = cp1

						if (uniqueIntersections.length === 1) {
							console.log('here', res, uniqueIntersections)
							const matchRest = inter.filter(curr => curr.includes(...uniqueIntersections))

							matchRest.forEach(curr => {
								const [p1, p2] = curr,
								cp1 = this.primaryCoords[p1],
								cp2 = this.primaryCoords[p2]
			
								this.primaryCoords[p1] = cp2
								this.primaryCoords[p2] = cp1
							})
						}
					})
				}

				
			console.log(uniqueIntersections)

			}
			
			const isDisjoint = this.intersect.every(curr => !curr.result)
			console.log({isDisjoint})
			// return !isDisjoint ? this.test() : false
			
		},

		
		
		changeSegments() {
			if (Object.keys(this.primaryCoords).length <= 2) return
			const clean = this.intersect.filter(f => !f.result).map(m => m.points)
			const dbl = this.intersect
				.filter(c => c.result)
				.map(c => c.points)
				.flat()
				.reduce((acc, curr, i, arr) => {
					const isDouble = arr.some((el, ii) => i !== ii && curr === el)
					if (isDouble) acc.push(curr)

					return acc
				}, [])

			const epicenter = Array.from(new Set(dbl))

			if (clean.length) {

				if (this.counter > 1 && this.counter !== null) {
					console.log('pizda', this.intersect, this.counter)

					if (epicenter.length) {
						if (epicenter.length === 1) {
							console.log('epic len = 1')
						} else {
							console.log('pizda > 1', epicenter)
							if (epicenter.length) {
								// setTimeout(() => {
									const [p1, p2] = epicenter.splice(0, 2)
									const cp1 = this.primaryCoords[p1]
									const cp2 = this.primaryCoords[p2]
									console.log(p1, p2, cp1, cp2)
									this.primaryCoords[p1] = cp2
									this.primaryCoords[p2] = cp1
									this.counter = null
									this.changeSegments()
								// }, 1000)
							}
						}
					} else {
						console.log('epicenter []')
					}
				}

				// setTimeout(() => {
					console.log(this.intersect)
	
					clean.forEach(cl => {
						const [p1, p2] = cl
						const cp1 = this.primaryCoords[p1]
						const cp2 = this.primaryCoords[p2]
						console.log(cp1, cp2, '???')
						this.primaryCoords[p1] = cp2
						this.primaryCoords[p2] = cp1
					})
					
					console.log('here')
					const isIntersect = !this.intersect.every(curr => !curr.result)
					console.log(isIntersect)
					this.counter++
					return isIntersect ? this.changeSegments() : false

				// }, 1000)
			} else {
				console.log('empty')



				if (epicenter.length === 1) {
					console.log('epic len = 1')
				} else {
					let x = () => {
						console.log('call x')
						// setTimeout(() => {
							epicenter.forEach(curr => {
								const [p1, p2] = epicenter.splice(0, 2)
								const cp1 = this.primaryCoords[p1]
								const cp2 = this.primaryCoords[p2]
								console.log(p1, p2)
								this.primaryCoords[p1] = cp2
								this.primaryCoords[p2] = cp1
							})
							
							return this.intersect.map(curr => curr.result).includes(false) ? this.changeSegments() : x()
						// }, 1000)
					}

					x()
				}
			}
		}
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
