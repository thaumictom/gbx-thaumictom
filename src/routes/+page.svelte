<script lang="ts">
	import Icon from '@iconify/svelte';
	import { CGameCtnChallenge, CGameCtnReplayRecord, GBX } from 'gbx';
	import pkg from 'tm-essentials';
	const { TextFormatter, Time } = pkg;
	import InfoBlock from './InfoBlock.svelte';
	import SubContent from './SubContent.svelte';

	// Constants
	const maxFiles = 15;
	const allowedFileTypes = '.Map.Gbx,.Challenge.Gbx,.Replay.Gbx,.Ghost.Gbx';

	let files: ProcessedFile[] = [];

	type ProcessedFile = {
		file: File;
		parseTime: number;
		detail: {
			gbx: GBX<CGameCtnChallenge | CGameCtnReplayRecord>;
			parsedGbx: CGameCtnChallenge | CGameCtnReplayRecord;
		};
	};

	// Unique files
	const addUniqueFiles = (newFiles: File[]) => {
		const existingFileNames = new Set(files.map((file: ProcessedFile) => file.file.name));
		const uniqueNewFiles = newFiles.filter(({ name }) => !existingFileNames.has(name));
		const trimmedFiles = uniqueNewFiles.splice(0, maxFiles - files.length);
		const processedFiles = trimmedFiles.map(async (file): Promise<ProcessedFile> => {
			const arrayBuffer = await file.arrayBuffer();
			const buffer = Array.from(new Uint8Array(arrayBuffer));

			const t0 = performance.now();

			const gbx = new GBX<CGameCtnChallenge | CGameCtnReplayRecord>(buffer, 3);
			const parsedGbx = await gbx.parse();

			const t1 = performance.now();

			return {
				file,
				parseTime: t1 - t0,
				detail: {
					gbx,
					parsedGbx
				}
			};
		});

		Promise.all(processedFiles).then((resolvedFiles) => {
			selectedFile = resolvedFiles[0];
			files = [...files, ...resolvedFiles];
		});
	};

	// Drag and drop operations
	const handleFileInput = (event: Event) => {
		const input = event.target as HTMLInputElement;
		if (input.files) {
			addUniqueFiles(Array.from(input.files) as File[]);
		}
	};

	const handleDrop = (event: DragEvent) => {
		dragging = false;
		if (event.dataTransfer?.files) {
			addUniqueFiles(Array.from(event.dataTransfer.files) as File[]);
		}
	};

	let hover: boolean, dragging: boolean;
	let dragRefCounter: number = 0;
	let selectedFile: ProcessedFile | null = null;
</script>

<div class="flex gap-4">
	<div
		data-selected={!selectedFile || null}
		class="flex w-sm shrink-0 flex-col gap-2 data-selected:w-full"
	>
		<div>
			<input
				hidden
				onchange={handleFileInput}
				id="file-upload"
				accept={allowedFileTypes}
				multiple={maxFiles > 1}
				type="file"
			/>
			<label
				for="file-upload"
				onmouseenter={() => (hover = true)}
				onmouseleave={() => (hover = false)}
				ondragenter={(event) => {
					event.preventDefault();
					dragRefCounter++;
					dragging = true;
				}}
				ondragleave={(event) => {
					event.preventDefault();
					dragRefCounter--;
					if (dragRefCounter === 0) dragging = false;
				}}
				ondragover={(event) => event.preventDefault()}
				ondrop={(event) => {
					event.preventDefault();
					handleDrop(event);
				}}
			>
				<div
					data-hover={hover || dragging || null}
					class="group flex cursor-pointer items-center justify-center gap-6 border-4 border-dashed p-8 data-hover:border-indigo-500"
				>
					<Icon icon="pixelarticons:cloud-upload" class="size-12" />

					<div class="text-center">
						Click to <strong>choose files</strong>
						or<br /> <strong>drag and drop</strong> them here
					</div>
				</div>
			</label>
		</div>
		{#each files as file}
			<div class="flex items-center">
				<button
					onclick={() => (selectedFile = file)}
					data-selected={selectedFile === file || null}
					class="bg-muted grow cursor-pointer truncate border-l-4 py-2 pl-4 text-left shadow-sm data-selected:border-indigo-500"
					title={TextFormatter.deformat(file.file.name)}
				>
					<div class="truncate pl-2">{TextFormatter.deformat(file.file.name)}</div>
				</button>
				<div class="bg-muted flex h-full items-center px-1">
					<button
						onclick={() => {
							if (selectedFile === file) selectedFile = null;
							files = files.filter((f) => f !== file);
						}}
						class="hover:bg-secondary cursor-pointer p-1"
					>
						<Icon icon="pixelarticons:close" class="size-6" />
					</button>
				</div>
			</div>
		{/each}
		{#if files.length}
			<div class="text-muted-fg text-right text-sm font-medium">
				{files.length} / {maxFiles} files
			</div>
		{:else}
			<div class="text-muted-fg text-right text-sm font-medium">
				Support files: {allowedFileTypes.split(',').join(', ')}
			</div>
		{/if}
	</div>
	{#if selectedFile}
		{@const {
			file,
			parseTime,
			detail: { gbx, parsedGbx }
		} = selectedFile}
		<div class="flex flex-1 shrink flex-col gap-4 overflow-auto">
			<div class="bg-muted flex items-center justify-between gap-4 p-4 shadow-sm">
				<div>
					<div class="text-lg">{TextFormatter.deformat(file.name)}</div>
					<div class="text-secondary-fg text-xs font-medium">
						classId: {gbx.wrappedClassId ? gbx.wrappedClassId : gbx.classId}
					</div>
				</div>
				<div class="text-right">
					<div>{(file.size / 1024 / 1024).toFixed(2)} MiB</div>
					<div class="text-secondary-fg text-xs font-medium">
						parsed in {parseTime.toFixed(2)} ms
					</div>
				</div>
			</div>
			<div class="flex flex-col gap-2 border-l-4 p-2 pl-4">
				{#each Object.entries(parsedGbx) as [key, value]}
					{#if !/^\d/.test(key)}
						{#if value instanceof GBX}
							<InfoBlock title={key} type={typeof value} gbx>
								{#await value.parse()}
									<div>Loading data...</div>
								{:then content}
									<SubContent {content}></SubContent>
								{/await}
							</InfoBlock>
						{:else}
							<InfoBlock title={key} type={typeof value}>
								{JSON.stringify(value)}
							</InfoBlock>
						{/if}
					{/if}
				{/each}
			</div>
		</div>
	{/if}
</div>
