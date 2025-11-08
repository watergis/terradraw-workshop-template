<script lang="ts">
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />

	<style>
		html,
		body {
			margin: 0;
			padding: 0;
			min-height: 100vh;
			min-height: -webkit-fill-available;
		}

		html {
			height: -webkit-fill-available;
		}
	</style>
</svelte:head>

{@render children()}
