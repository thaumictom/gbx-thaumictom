<script lang="ts">
	import { Collapsible } from 'bits-ui';

	export let title;
	export let type;
	export let gbx: boolean = false;
</script>

<Collapsible.Root>
	<Collapsible.Trigger
		class="disabled:text-muted-fg w-full text-left not-disabled:cursor-pointer data-[state=open]:text-indigo-500"
		disabled={type === 'undefined'}
	>
		<div class="font-bold">{title}{gbx ? '...' : ''}</div>
	</Collapsible.Trigger>
	<Collapsible.Content>
		<slot></slot>
	</Collapsible.Content>
</Collapsible.Root>
