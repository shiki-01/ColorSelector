<script lang="ts">
	import '../app.css';
	let { children } = $props();
</script>

<svelte:head>
    <title>Color Selector</title>
</svelte:head>

<svelte:document
    on:touchmove={e => e.preventDefault()}
    on:touchstart={e => e.preventDefault()}
/>

{@render children()}

<style>
    :global(html) {
        overflow: hidden;
        touch-action: none;
        -webkit-user-select: none;
        -webkit-touch-callout: none;
        user-select: none;
    }
</style>