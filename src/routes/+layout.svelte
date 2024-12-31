<script lang="ts">
	import Header from '$lib/components/layout/Header.svelte';
	import '../app.css';
	let { children } = $props();
</script>

<div class="flex h-screen flex-col">
	<Header title="Untitled Spreadsheet" />
	<main class="flex h-0 grow flex-col">
		{@render children()}
	</main>
	<footer></footer>
</div>
