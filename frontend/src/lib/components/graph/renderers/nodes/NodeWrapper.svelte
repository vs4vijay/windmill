<script lang="ts">
	import DarkModeObserver from '$lib/components/DarkModeObserver.svelte'
	import { Handle, Position } from '@xyflow/svelte'

	export let enableSourceHandle: boolean = true
	export let enableTargetHandle: boolean = true
	export let offset: number = 0

	let darkMode: boolean = false
</script>

<DarkModeObserver bind:darkMode />

<div class="relative shadow-md" style={`margin-left: ${offset}px;`}>
	<slot {darkMode} />
</div>

{#if enableSourceHandle}
	<Handle type="source" position={Position.Bottom} style={`margin-left: ${offset / 2}px;`} />
{/if}

{#if enableTargetHandle}
	<Handle type="target" position={Position.Top} style={`margin-left: ${offset / 2}px;`} />
{/if}
