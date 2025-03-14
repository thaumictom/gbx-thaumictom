<script>
	import Icon from '@iconify/svelte';
	import { Dialog } from 'bits-ui';
	import { onMount } from 'svelte';

	let hash = $state('');
	let open = $state(false);
	let once = false;

	$effect(() => {
		if (open && !once) {
			once = true;
			fetch('https://api.github.com/repos/thaumictom/gbx-thaumictom/commits')
				.then((res) => res.json())
				.then((data) => {
					hash = data[0].sha.slice(0, 7);
				});
		}
	});
</script>

<Dialog.Root bind:open>
	<Dialog.Trigger class="hover:bg-secondary hover:text-secondary-fg cursor-pointer p-1">
		<Icon icon="pixelarticons:info-box" class="size-6" />
	</Dialog.Trigger>
	<Dialog.Portal>
		<Dialog.Overlay class="bg-default/50 fixed inset-0 z-20 backdrop-blur-sm"></Dialog.Overlay>
		<Dialog.Content class="fixed top-1/2 left-1/2 z-20 max-w-md -translate-1/2 ">
			<div class="bg-default flex flex-col gap-4 border p-4">
				<div class="flex items-center justify-between">
					<div class="font-display text-lg">Information</div>
					<Dialog.Close class="hover:bg-secondary hover:text-secondary-fg cursor-pointer p-1">
						<Icon icon="pixelarticons:close" class="size-6" />
					</Dialog.Close>
				</div>
				<div class="flex flex-col gap-4">
					<div>
						gbx.thaumictom.de is a free read-only tool to analyze Trackmania files using <a
							class="font-semibold text-indigo-500"
							href="https://github.com/thaumictom/gbx-ts">gbx-ts</a
						>
						and
						<a class="font-semibold text-indigo-500" href="https://github.com/thaumictom/lzo-ts"
							>lzo-ts</a
						>
						hosted by thaumictom. This project is licensed under GPLv3.
					</div>
					<div>
						Currently supported:
						<ul class="list-['-'] pl-2 *:pl-2">
							<li>CGameCtnChallenge (.Map.Gbx, .Challenge.Gbx)</li>
							<li>CGameCtnReplayRecord (.Replay.Gbx)</li>
							<li>CGameCtnGhost (.Ghost.Gbx)</li>
						</ul>
					</div>
					<div>
						If you need a more sophisticated tools, please consider using <a
							class="font-semibold text-indigo-500"
							href="https://gbx.tools/"
						>
							gbx.tools</a
						> by BigBang1112.
					</div>
					<div class="flex justify-between">
						<div class="text-muted-fg text-sm">{hash}</div>
						<div class="text-muted-fg text-right text-sm">© 2025 thaumictom.de</div>
					</div>
				</div>
			</div>
		</Dialog.Content>
	</Dialog.Portal>
</Dialog.Root>
