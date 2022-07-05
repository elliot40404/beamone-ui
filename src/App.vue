<script setup>
	import { onMounted } from 'vue';
	import { RouterView } from 'vue-router';
	onMounted(() => {
		const iconsEsm = document.createElement('script');
		iconsEsm.src =
			'https://unpkg.com/ionicons@5.5.2/dist/ionicons/ionicons.esm.js';
		iconsEsm.type = 'module';
		const icon = document.createElement('script');
		icon.src = 'https://unpkg.com/ionicons@5.5.2/dist/ionicons/ionicons.js';
		icon.noModule = true;
		document.head.appendChild(iconsEsm);
		document.head.appendChild(icon);
	});
</script>

<template>
	<RouterView />
</template>

<style lang="scss">
	@import '@/assets/base.scss';
</style>
