<script lang="ts">
	import '../app.css';
	let { children } = $props();
</script>

<svelte:head>
    <title>Color Selector</title>
</svelte:head>

{@render children()}
