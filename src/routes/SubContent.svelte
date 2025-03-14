<script lang="ts">
	import InfoBlock from './InfoBlock.svelte';

	export let content;
</script>

<div class="flex flex-col gap-2 border-l-4 p-2 pl-4">
	{#each Object.entries(content) as [key, value]}
		{#if !/^\d/.test(key)}
			<InfoBlock title={key} type={typeof value}>
				{JSON.stringify(value)}
			</InfoBlock>
		{/if}
	{/each}
</div>
