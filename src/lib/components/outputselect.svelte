<script lang="ts">
	import Check from 'lucide-svelte/icons/check';
	import ChevronsUpDown from 'lucide-svelte/icons/chevrons-up-down';
	import { tick } from 'svelte';
	import * as Command from '$lib/components/ui/command/index.js';
	import * as Popover from '$lib/components/ui/popover/index.js';
	import { Button } from '$lib/components/ui/button/index.js';
	import { cn } from '$lib/utils.js';
	import { toast } from 'svelte-sonner';

	let { value = $bindable() }: { value: string } = $props();

	let devices = $state<MediaDeviceInfo[] | null>(null);

	let open = $state(false);
	let triggerRef = $state<HTMLButtonElement>(null!);

	const selectedValue = $derived(devices?.find((d) => d.deviceId === value)?.label);

	// We want to refocus the trigger button when the user selects
	// an item from the list so users can continue navigating the
	// rest of the form with the keyboard.
	function closeAndFocusTrigger() {
		open = false;
		tick().then(() => {
			triggerRef.focus();
		});
	}

	async function handleOpenChange(newOpen: boolean) {
		if (devices !== null) {
			open = newOpen;
			return;
		}
		if (!newOpen) return;
		try {
			await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
			devices = (await navigator.mediaDevices.enumerateDevices()).filter(
				(d) => d.kind === 'audiooutput'
			);

			open = newOpen;
		} catch (e) {
			let message;
			if (e instanceof Error) {
				message = e.message;
			} else {
				message = `${e}`;
			}
			toast.error(message);
		}
	}
</script>

<Popover.Root bind:open onOpenChange={handleOpenChange}>
	<Popover.Trigger bind:ref={triggerRef}>
		{#snippet child({ props })}
			<Button
				variant="outline"
				class="justify-between"
				{...props}
				role="combobox"
				aria-expanded={open}
			>
				{selectedValue || 'Select an audio output...'}
				<ChevronsUpDown class="ml-2 size-4 shrink-0 opacity-50" />
			</Button>
		{/snippet}
	</Popover.Trigger>
	<Popover.Content class="max-w-[300px] p-0" align="start">
		<Command.Root>
			<Command.Input placeholder="Search outputs..." />
			<Command.List>
				<Command.Empty>No outputs found.</Command.Empty>
				<Command.Group>
					{#if devices}
						{#each devices as device}
							<Command.Item
								value={device.deviceId}
								onSelect={() => {
									value = device.deviceId;
									closeAndFocusTrigger();
								}}
							>
								<Check class={cn('mr-2 size-4', value !== device.deviceId && 'text-transparent')} />
								{device.label}
							</Command.Item>
						{/each}
					{:else}
						<Command.Item disabled>Allow access to your microphone to view outputs.</Command.Item>
					{/if}
				</Command.Group>
			</Command.List>
		</Command.Root>
	</Popover.Content>
</Popover.Root>
