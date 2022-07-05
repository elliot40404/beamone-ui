<script setup>
	import { computed } from 'vue';

	const props = defineProps({
		custom: {
			type: Boolean,
		},
		color: {
			type: String,
		},
		icon: {
			type: String,
		},
		title: {
			type: String,
		},
		value: {
			type: Number,
		},
	});
	const val = computed(() => {
		return props.value;
	});
</script>
<template>
	<div class="default" v-if="custom">
		<slot> Custom hello </slot>
	</div>
	<div class="default default-card" v-else>
		<div class="card">
			<h3>{{ title.toUpperCase() }}</h3>
			<h1>{{ val.toLocaleString() }}</h1>
			<ion-icon :name="icon"></ion-icon>
		</div>
	</div>
</template>

<style lang="scss" scoped>
	.default {
		border-radius: 0.5rem;
		background-color: #fff;
		box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.1);
	}
	.default-card {
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.card {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 1rem;
		gap: 0.8rem;
	}
	h3 {
		font-weight: 500;
		color: rgb(69, 69, 69);
	}
	ion-icon {
		font-size: 1.5rem;
		color: rgb(0, 0, 0);
		margin-bottom: 0.5rem;
		color: v-bind(color);
	}
</style>
