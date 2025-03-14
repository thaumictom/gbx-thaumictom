<script lang="ts">
	import { ModeWatcher, toggleMode } from 'mode-watcher';
	import '../app.css';
	import Icon from '@iconify/svelte';
	import InfoButton from './InfoButton.svelte';
	let { children } = $props();
</script>

<ModeWatcher lightClassNames={['light']} defaultMode={'dark'}></ModeWatcher>

<div class="bg-default text-foreground h-full min-h-screen font-sans">
	<div class="container border-b">
		<div class="relative">
			<div
				class="to-default absolute top-0 left-0 h-full w-full from-indigo-500/20 to-70% dark:bg-radial-[at_bottom]"
			></div>
			<div class="relative z-1 flex items-center justify-between py-4">
				<div>
					<a class="font-display text-xl" href="/">gbx.thaumictom.de</a>
				</div>
				<div class="flex gap-1">
					<InfoButton></InfoButton>
					<button
						onclick={toggleMode}
						class="hover:bg-secondary hover:text-secondary-fg z-1 cursor-pointer p-1"
					>
						<Icon icon="pixelarticons:sun" class="size-6 dark:hidden" />
						<Icon icon="pixelarticons:moon-stars" class="size-6 not-dark:hidden" />
					</button>
					<a
						href="https://github.com/thaumictom/gbx-thaumictom"
						class="hover:bg-secondary hover:text-secondary-fg p-1"
					>
						<Icon icon="pixelarticons:github" class="size-6" />
					</a>
				</div>
			</div>
		</div>
	</div>
	<div class="container py-16">
		{@render children()}
	</div>
</div>
