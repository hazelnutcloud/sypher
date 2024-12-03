<script lang="ts">
	import '../app.css';
	import { Toaster } from '$lib/components/ui/sonner';
	import * as Card from '$lib/components/ui/card';

	let { children } = $props();
</script>

<Toaster />
<div class="flex min-h-screen flex-col items-center justify-center p-8">
	<Card.Root class="w-full max-w-screen-sm">
		<Card.Content class="flex flex-col items-start gap-4">{@render children()}</Card.Content>
	</Card.Root>
</div>
