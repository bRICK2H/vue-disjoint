<template>
	<div class="interface container__interface">
		<button v-if="!isCoords"
			@click="addCoords"
		>Задать входные данные</button>
		<button v-if="isCoords"
			@click="deleteField"
		>
			Удалить все
		</button>

			<div class="container-points">
				<transition-group name="points" tag="div">
					<div class="list-points" 
						v-for="(points, key) of coords"
						:key="`point-${key}`"
						@input="inputValue($event, key)"
						@click="updatePrevValue($event.target.value)"
					>
						<span>{{ key }}</span>
						<div class="item-point">
							<span>x:</span>
							<input class="input-point item-point__input-point"
								:class="{ 'input-point--double': isDoubleCoords && doubleCoords.includes(key) || isLimitValue && limitKeyPoint === key && limitTypePoint === 'x' }"
								type="text"
								:value="points.x"
								data-type="x"
							>
							<span>y:</span>
							<input class="input-point item-point__input-point"
								:class="{ 'input-point--double': isDoubleCoords && doubleCoords.includes(key) || isLimitValue && limitKeyPoint === key && limitTypePoint === 'y' }"
								type="text"
								:value="points.y"
								data-type="y"
							>
							<button class="btn-clear input-point__btn-clear" @click="deteleValue(key)"></button>
						</div>
					</div>
				</transition-group>
			</div>

		<!-- <template v-if="isCoords"> -->
			<div class="container-buttons">
				<button v-if="isCoords"
					@click="addCoords"
				>
					Добавить точку
				</button>
				<button @click="updateValue">
					Обработать
				</button>
			</div>
		<!-- </template> -->

	</div>
</template>

<script>
export default {
	name: 'cInterface',
	props: {
		points: {
			type: Array,
			default: () => ({})
		}
	},
	data: () => ({
		coords: {},
		prevValue: null,
		doubleCoords: [],
		isDoubleCoords: false,
		isLimitValue: false,
		limitTypePoint: '',
		limitValue: 0
	}),
	computed: {
		isCoords() {
			return !!Object.keys(this.coords).length
		}
	},
	methods: {
		addCoords() {
			const newKeyPoint = !!Object.keys(this.coords).length
				? Math.max(...Object.keys(this.coords)) + 1 : 1
				
			this.$set(this.coords, newKeyPoint, { x: 0, y: 0 })
		},
		inputValue(e, keyPoint) {
			const { target, target: { dataset: { type } } } = e

			if (target.value.length > 3) {
				target.value = this.prevValue
				this.isLimitValue = true
				this.limitKeyPoint = keyPoint
				this.limitTypePoint = type

				setTimeout(() => {
					this.limitValue = 0
					this.isLimitValue = false
					this.limitTypePoint = ''
				}, 300)
				
				return
			}
			target.value = this.prevValue = target.value.replace(/[^\d]/, '')
			this.$set(this.coords[keyPoint], type, +target.value)
		},
		updatePrevValue(value) {
			this.prevValue = value
		},
		updateValue() {
			const coords = Object.entries(this.coords)
			const doubleCoords = coords
				.filter((curr, i) => {
					const [, { x: cx, y: cy }] = curr

					return coords.some((curr, j) => {
						const [, { x, y }] = curr
						
						return cx === x && cy === y && i !== j
					})
				})

			if (doubleCoords.length) {
				this.doubleCoords = doubleCoords.map(curr => {
					const [keyPoint] = curr
					return keyPoint
				})

				this.isDoubleCoords = true
				setTimeout(() => {
					this.doubleCoords = []
					this.isDoubleCoords = false
				}, 300)

				return false
			}

			this.prevValue = null
			this.$emit('create-disjoint', this.contertValue(this.coords))
		},
		deteleValue(point) {
			this.$delete(this.coords, point)
		},
		deleteField() {
			this.coords = {}
		},
		contertValue(coords) {
			return Object.entries(coords).reduce((acc, curr) => {
				const [point, { x, y }] = curr

				if(!acc.length) acc = []
				acc.push({ [point]:  [x, y]})

				return acc
			}, [])
		}
	},
	watch: {
		points: {
			immediate: true,
			handler(points) {
				this.coords =  points.reduce((acc, curr) => {
					const [[point, [x, y]]] = Object.entries(curr)
					acc[point] = {x, y}
					
					return acc
				}, {})
			}
		}
	}
}
</script>

<style lang="scss">
	.interface {
		width: 30rem;
		height: 100%;
		border: 1px solid red;
	}

	.container-points {
		max-height: calc(100% - 10rem);
		overflow-y: auto;
	}

	.list-points,
	.item-point {
		display: flex;
		align-items: center;
	}
	.item-point {
		justify-content: center;

		&__input-point {
			margin: .5rem;
		}
	}

	.input-point {
		width: 100%;
		text-align: center;
		padding: .5rem 0;
		position: relative;
		outline: none;
		border: none;
		border-radius: .5rem;
		border: .1rem solid #222;

		&--double {
			animation: dobule .3s ease;
			@keyframes dobule {
				0% { left: .2rem; border: .1rem solid tomato; }
				25% { left: -.2rem; border: .1rem solid tomato; }
				50% { left: .2rem; border: .1rem solid tomato; }
				75% { left: -.2rem; }
			}
		}
	}

	.btn-clear {
		// width: 50px;
		height: 25px;
		flex: 1 0 25px;
		border: none;
		border-radius: 50%;
		background-image: url('../assets/img/svg/del-all.svg');
		background-color: #fff;
		background-size: cover;
		background-repeat: no-repeat;
		background-position: 50% 50%;

		&:hover {
			cursor: pointer;
			filter: brightness(.8);
		}
	}

	.points-leave-active {
		animation: leave-points .5s;
		@keyframes leave-points {
			100% { opacity: 0; transform: translate(-100%)}
		}
	}
</style>