<template>
	<div class="interface container__interface">
		<button v-if="!isPrimary">Задать входные данные</button>

		<div class="container-points" 
			v-for="(points, key) of coords"
			:key="key"
			@input="inputValue($event, key)"
			@click="updatePrevValue($event.target.value)"
		>
			<span>{{ key }}</span>
			<div class="box-points">
				<span>x:</span>
				<input class="input-point box-points__input-point"
					type="text"
					:value="points.x"
					data-type="x"
				>
				<span>y:</span>
				<input class="input-point box-points__input-point"
					type="text"
					:value="points.y"
					data-type="y"
				>
				<button @click="deteleValue(key)">del</button>
			</div>
		</div>

		<!-- <template v-if="isPrimary"> -->
			<button>Добавить точку</button>
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
		isPrimary() {
			return !!this.points.length
		}
	},
	methods: {
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

	.container-points,
	.box-points {
		display: flex;
		align-items: center;
	}
	.box-points {
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