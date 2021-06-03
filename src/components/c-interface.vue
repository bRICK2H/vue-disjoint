<template>
	<div class="interface container__interface">

		<c-warning
			:isError="isDoubleCoords || isLimitValue"
			@close-warning="closeWarinig"
		>
			{{ warning }}
		</c-warning>
		
		<div class="container-buttons">
			<transition name="btn">
				<button v-if="!isCoords"
					class="btn btn--entry container-buttons__btn"
					@click="addCoords"
				>
					Входные данные
				</button>
			</transition>
			<transition name="btn">
				<button v-if="isCoords"
					@click="deleteField"
					class="btn btn--all-del container-buttons__btn"
				>
					Удалить все
				</button>
			</transition>
		</div>

		<div class="container-points">
			<transition-group name="points" tag="div">
				<div class="list-points" 
					v-for="(points, key) of coords"
					:key="`point-${key}`"
					@input="inputValue($event, key)"
					@click="updatePrevValue($event)"
				>
					<span class="item-key-point container-points__item-key-point">{{ key }}</span>
					<div class="item-point">
						<span class="type-point item-point__type-point">x:</span>
						<input class="input-point item-point__input-point"
							:class="{ 'input-point--double': isDoubleCoords && doubleCoords.includes(key) || isLimitValue && limitKeyPoint === key && limitTypePoint === 'x' }"
							type="text"
							:value="points.x"
							data-type="x"
						>
						<span class="type-point item-point__type-point">y:</span>
						<input class="input-point item-point__input-point"
							:class="{ 'input-point--double': isDoubleCoords && doubleCoords.includes(key) || isLimitValue && limitKeyPoint === key && limitTypePoint === 'y' }"
							type="text"
							:value="points.y"
							data-type="y"
						>
						<button class="btn-clear input-point__btn-clear"
							@click="deteleValue(key)"
							title="Удалить"
						></button>
					</div>
				</div>
			</transition-group>
		</div>

		<transition name="btn">
			<div class="container-buttons"
				v-if="isCoords"
			>
				<button class="btn btn--add container-buttons__btn"
					@click="addCoords"
				>
					Добавить точку
				</button>
				<transition name="btn">
					<button v-if="isCoords > 1"
						class="btn btn--process container-buttons__btn"
						@click="updateValue"
					>
						Обработать
					</button>
				</transition>
			</div>
		</transition>

	</div>
</template>

<script>
import cWarning from './c-warning'

export default {
	name: 'cInterface',
	components: {
		cWarning
	},
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
		limitValue: 0,
		warning: 'Ошибка'
	}),
	computed: {
		isCoords() {
			return Object.keys(this.coords).length
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
				
				return
			}
			target.value = this.prevValue = target.value.replace(/[^\d]/, '')
			this.$set(this.coords[keyPoint], type, +target.value)
		},
		updatePrevValue(e) {
			const { target } = e
			
			this.prevValue = target.value
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
				this.isDoubleCoords = true
				this.doubleCoords = doubleCoords.map(curr => {
					const [keyPoint] = curr
					return keyPoint
				})

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
		},
		closeWarinig() {
			this.isDoubleCoords = this.isLimitValue = false
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
		},
		isDoubleCoords() {
			this.warning = `Координаты ${this.doubleCoords.join(', ')} дублируются!`
		},
		isLimitValue(val) {
			console.log('isLimit', val, this.limitTypePoint)
		}
	}
}
</script>

<style lang="scss">
	.interface {
		flex: 0 1 30rem;
    	min-width: 30rem;
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center
	}
	.container-points {
		max-height: calc(100% - 12rem);
		overflow-y: auto;

		&__item-key-point {
			margin-right: 1rem;
		}
	}
	.list-points {
		position: relative;

		&::after {
			content: '';
			width: 100%;
			height: .1rem;
			background-color: #aaa;
			position: absolute;
			bottom: 0;
			left: 0;
		}
	}
	.list-points,
	.item-point {
		display: flex;
		align-items: center;
	}
	.item-key-point {
		min-width: 3rem;
		font-size: 1.7rem;
		font-weight: 400;
		text-align: right;
	}
	.item-point {
		justify-content: center;

		&__input-point {
			margin: .5rem;
		}
		&__type-point {
			margin: 1rem;
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
	.type-point {
		font-size: 1.7rem;
		font-weight: 400;
	}
	.btn-clear {
		height: 22px;
		flex: 1 0 22px;
		margin: 0 1rem;
		border: none;
		outline: none;
		border-radius: 50%;
		background-image: url('../assets/img/svg/del.svg');
		background-color: #fff;
		background-size: cover;
		background-repeat: no-repeat;
		background-position: 50% 50%;

		&:hover {
			cursor: pointer;
			filter: brightness(.8);
		}
	}

	.container-buttons {
		display: flex;
		justify-content: center;

		&__btn {
			flex: 0 1 50%;
			margin: 1rem .5rem;
		}
	}

	.btn {
		font-size: 1.6rem;
		padding: 1rem 0;
		border: none;
		outline: none;
		border-radius: .3rem;
		user-select: none;
		color: #fff;
		cursor: pointer;
		transition: .2s;

		&--all-del {
			margin-left: auto;
			background-color: #dc3545;

			&:hover {
				background-color: rgba(220, 53, 70, .8);
			}
		}
		&--all-del,
		&--entry {
			flex: 0 1 calc(50% - 1rem)
		}
		&--entry {
			margin-right: auto;
		}
		&--entry,
		&--add {
			background-color: #28a745;

			&:hover {
				background-color: rgba(40, 167, 70, .8);
			}
		}
		&--process {
			background-color: #007bff;

			&:hover {
				background-color: rgba(0, 123, 255, .8);
			}
		}

		&:active {
			transform: scale(.95);
			box-shadow: 0 .2rem .6rem #222;
		}
	}

	.points-leave-active {
		animation: leave-point .5s;
		@keyframes leave-point {
			100% { opacity: 0; transform: translate(-100%)}
		}
	}
	.points-enter-active {
		animation: enter-point .5s;
		@keyframes enter-point {
			0% { opacity: 0; transform: translate(-100%)}
		}
	}
	.btn-leave-active {
		animation: leave-btn .5s;
		@keyframes leave-btn {
			100% { transform: perspective(300px) rotateX(90deg) }
		}
	}
	.btn-enter-active {
		animation: enter-btn .5s;
		@keyframes enter-btn {
			0% { transform: perspective(300px) rotateX(90deg) }
		}
	}
</style>