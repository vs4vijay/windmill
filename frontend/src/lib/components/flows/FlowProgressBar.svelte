<script lang="ts">
	import { type Job } from '$lib/gen'
	import ProgressBar from '../progressBar/ProgressBar.svelte'

	export let job: Job | undefined = undefined
	export let currentSubJobProgress: number | undefined = undefined

	let error: number | undefined = undefined
	let index = 0
	let subIndex: number | undefined = undefined
	let subLength: number | undefined = undefined
	let length = 1
	let nextInProgress = false
	let subIndexIsPercent: boolean = false

	$: if (job) updateJobProgress(job)

	function updateJobProgress(job: Job) {
		const modules = job?.flow_status?.modules
		if (!modules?.length) {
			return
		}

		let subStepIndex: undefined | number = undefined
		let subStepLength: undefined | number = undefined
		let newError: number | undefined = undefined
		let newNextInProgress = false

		let maxDone = Math.max(job?.flow_status?.step ?? 0, 0)
		if (modules.length > maxDone) {
			const nextModule = modules[maxDone]
			if (nextModule.type === 'InProgress') {
				newNextInProgress = true
			}
		}

		let module = modules[maxDone]
		if (module) {
			if (module.type === 'Failure' || (module.type === 'Success' && job['success'] === false)) {
				newError = maxDone
				maxDone = maxDone + 1
			}
		}		
		subIndexIsPercent = false;

		// Loop is still iterating
		if (module?.iterator) {
			const stepIndex = module.iterator.index || 0
			const stepLength = module.iterator.itered?.length || 0
			if (module.iterator.index != undefined) {
				subStepIndex = stepIndex
				subStepLength = stepLength
			}
		} else if (module?.branchall) {
			subStepIndex = module.branchall.branch
			subStepLength = module.branchall.len
		} else if (module?.progress) {		
			const clamp = (num, min, max) => Math.min(Math.max(num, min), max)
			subStepIndex = clamp(module?.progress, subIndex ?? 0, 99)
			//                  Jitter protection >^^^^^^^^
			subStepLength = 100
			subIndexIsPercent = true;
			currentSubJobProgress = subStepIndex
		} else {
			currentSubJobProgress = undefined
		}

		error = newError
		subLength = subStepLength ? Math.max(subStepLength, 1) : undefined
		subIndex = subStepIndex
		length = Math.max(modules.length, 1)
		index = maxDone
		nextInProgress = newNextInProgress
	}

	let resetP: any

	export function reset() {
		resetP?.()
		error = undefined
		subIndex = undefined
		subLength = undefined
		length = 1
		index = 0
	}
</script>

<ProgressBar
	bind:resetP
	{length}
	{index}
	{nextInProgress}
	{subLength}
	{subIndex}
	{error}
	bind:subIndexIsPercent
	class={$$props.class}
/>
