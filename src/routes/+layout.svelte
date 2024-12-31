<script lang="ts">
	import Header from '$lib/components/layout/Header.svelte';
	import '../app.css';
	let { children } = $props();
</script>

<Header title="Untitled Spreadsheet" />
<main>
	{@render children()}
</main>
<footer></footer>
