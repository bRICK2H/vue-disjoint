<template>
	<div id="app">
		<div class="container">
			<div class="chart container__chart">
				<div class="chart__list chart__list-1">
					<span class="chart__item chart__item-1"></span>
					<span class="chart__item chart__item-2"></span>
				</div>
				<div class="chart__list chart__list-2">
					<span class="chart__item"
						:style="{ top: '400px', left: '400px' }"
					></span>
					<span class="chart__item"
						:style="{ top: '400px', left: '100px' }"
					></span>
				</div>
				<div class="chart__list chart__list-3">
					<span class="chart__item"
						:style="{ top: '400px', left: '100px' }"
					></span>
					<span class="chart__item"
						:style="{ top: '100px', left: '400px' }"
					></span>
				</div>
			</div>
		</div>
	</div>
</template>

<script>

export default {
	name: 'App',
	components: {},
	data: () => ({
		coords: {
			1: { x: 100, y: 100 },
			2: { x: 400, y: 400 },
			3: { x: 400, y: 100 },
			4: { x: 100, y: 400 },
		}
	})
}
</script>

<style lang="scss">
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
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
		width: 500px;
		height: 500px;
		border: 1px solid red;
		position: relative;

		&__list {
			width: 100%;
			height: 100%;
		}
		&__list-1 {
			// padding: 10px 40px 40px 10px;
		}
		// &__list-2 {
		// 	background: green;
		// }
		// &__list-3 {
		// 	background: blue;
		// }

		&__item {
			width: 100%;
			height: 100%;
			// position: absolute;
		}
		&__item-1 {
			display: inline-block;
			// border: 1px solid blue;
			background: blue;
			// clip-path: polygon(0% 0%, 100% 95%, 100% 100%);
			clip-path: polygon(
				10px 11px,
				calc(100% - 11px) calc(100% - 10px),
				calc(100% - 10px) calc(100% - 11px),
				11px 10px
			);
		}
	}
</style>
