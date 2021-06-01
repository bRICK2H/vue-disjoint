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
			<div class="list-points" 
				v-for="(points, key) of coords"
				:key="key"
				@input="inputValue($event, key)"
				@click="updatePrevValue($event.target.value)"
			>
				<span>{{ key }}</span>
				<div class="item-point">
					<span>x:</span>
					<input class="input-point item-point__input-point"
						type="text"
						:value="points.x"
						data-type="x"
					>
					<span>y:</span>
					<input class="input-point item-point__input-point"
						type="text"
						:value="points.y"
						data-type="y"
					>
					<button @click="deteleValue(key)">del</button>
				</div>
			</div>
		</div>

		<!-- <template v-if="isCoords"> -->
			<button v-if="isCoords"
				@click="addCoords"
			>
				Добавить точку
			</button>
			<button @click="updateValue">
				Обработать
			</button>
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
		prevValue: null
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
		inputValue(e, point) {
			const { target, target: { dataset: { type } } } = e

			if (target.value.length > 3) {
				target.value = this.prevValue
				return
			}
			target.value = this.prevValue = target.value.replace(/[^\d]/, '')
			this.$set(this.coords[point], type, +target.value)
		},
		updatePrevValue(value) {
			this.prevValue = value
		},
		updateValue() {
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
	}
</style>