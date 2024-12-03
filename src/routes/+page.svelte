<script lang="ts">
	import { browser } from '$app/environment';
	import Outputselect from '$lib/components/outputselect.svelte';
	import { Button } from '$lib/components/ui/button';
	import Input from '$lib/components/ui/input/input.svelte';
	import { toast } from 'svelte-sonner';

	let selectedDeviceId = $state('');
	let elevenLabsKey = $state(browser ? localStorage.getItem('elevenLabsKey') : null);
	let text = $state('Hello World!');
	let audio = $state<HTMLAudioElement>();

	$effect(() => {
		if (elevenLabsKey) {
			localStorage.setItem('elevenLabsKey', elevenLabsKey);
		}
	});

	$effect(() => {
		audio?.setSinkId(selectedDeviceId);
	});

	async function handlePlay() {
		if (!audio) return;

		if (!audio.paused) return;

		if (!elevenLabsKey) {
			toast.error('API key not set');
			return;
		}

		if (!text) {
			toast.error('Enter text');
			return;
		}

		const res = await fetch(
			`https://api.elevenlabs.io/v1/text-to-speech/cgSgspJ2msm6clMCkdW9?output_format=mp3_44100_128`,
			{
				headers: {
					'xi-api-key': elevenLabsKey,
					'content-type': 'application/json'
				},
				method: 'POST',
				body: JSON.stringify({
					text
				})
			}
		);

		if (!res.ok) {
			toast.error(`${res.status}: ${res.statusText}`);
			console.error(res);
			return;
		}

		const blob = await res.blob();
		const url = URL.createObjectURL(blob);

		audio.src = url;

		await audio.play();

		audio.onended = () => {
			URL.revokeObjectURL(url);
		};
	}
</script>

<audio bind:this={audio}></audio>

<span>Audio Output</span>
<Outputselect bind:value={selectedDeviceId} />
<span>ElevenLabs API Key</span>
<Input bind:value={elevenLabsKey} placeholder="sk_..." type="password" />
<span>Text</span>
<Input bind:value={text} placeholder="Hello World!" />
<div class="flex w-full justify-end">
	<Button onclick={handlePlay}>Play</Button>
</div>
