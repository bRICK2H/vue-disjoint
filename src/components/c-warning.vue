<template>
	<transition name="warning" v-if="isError">
		<div class="warning">
			<slot />
		</div>
	</transition>
</template>

<script>
export default {
	name: 'cWarning',
	props: {
		isError: {
			type: Boolean,
			default: false
		}
	},
	data: () => ({
		id: null
	}),
	watch: {
		isError() {
			clearTimeout(this.id)

			this.id = setTimeout(() => {
				this.$emit('close-warning')
			}, 2000)
		}
	}
}
</script>

<style lang="scss">
	.warning {
		max-width: 40rem;
		padding: 2rem;
		border-radius: .5rem;
		font-weight: 500;
		background-color: lightcoral;
		color: #fff;
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		z-index: 1000;
	}
	.warning-leave-active {
		animation: leave-warning 1s;
		@keyframes leave-warning {
			100% { transform: translate(-50%, -50%) translateY(-100vh) }
		}
	}
	.warning-enter-active {
		animation: enter-warning .5s;
		@keyframes enter-warning {
			0% { transform: translate(-50%, -50%) translateY(100vh) }
		}
	}
</style>