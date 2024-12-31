<script lang="ts">
	import LucideFileSpreadsheet from '~icons/lucide/file-spreadsheet';
	import { Button } from '../ui/button';
	let { title }: { title: string } = $props();
</script>

<span class="iconify" data-icon="solar:document-add-bold" data-inline="false"></span>
<header class="flex items-center gap-2 p-4">
	<LucideFileSpreadsheet class="size-10 text-primary" />
	<div>
		<h1 class="px-2 text-xl">{title}</h1>
		<nav>
			<Button size="sm" class="h-min px-2 py-1" variant="ghost">File</Button>
			<Button size="sm" class="h-min px-2 py-1" variant="ghost">Edit</Button>
			<Button size="sm" class="h-min px-2 py-1" variant="ghost">View</Button>
			<Button size="sm" class="h-min px-2 py-1" variant="ghost">Insert</Button>
			<Button size="sm" class="h-min px-2 py-1" variant="ghost">Format</Button>
			<Button size="sm" class="h-min px-2 py-1" variant="ghost">Data</Button>
			<Button size="sm" class="h-min px-2 py-1" variant="ghost">Tools</Button>
		</nav>
	</div>
</header>
