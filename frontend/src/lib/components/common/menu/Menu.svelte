<script lang="ts" context="module">
	let current: (() => void) | undefined = undefined
</script>

<script lang="ts">
	import { classNames } from '$lib/utils'
	import { onMount } from 'svelte'
	import { fade } from 'svelte/transition'

	export let noMinW = false
	export let show = false
	export let wrapperClasses = ''
	export let popupClasses = ''
	export let transitionDuration = 25
	let menu: HTMLDivElement

	type Alignment = 'start' | 'end' | 'center'
	type Side = 'top' | 'bottom'
	type Placement = `${Side}-${Alignment}`

	export let placement: Placement = 'bottom-start'

	function handleOutsideClick(event) {
		if (show && !menu.contains(event.target)) {
			show = false
			event.preventDefault()
			event.stopPropagation()
		}
	}

	function handleEscape(event) {
		if (show && event.key === 'Escape') {
			show = false
		}
	}

	function close() {
		show = false
	}

	onMount(() => {
		document.addEventListener('click', handleOutsideClick, false)
		document.addEventListener('keyup', handleEscape, false)

		return () => {
			document.removeEventListener('click', handleOutsideClick, false)
			document.removeEventListener('keyup', handleEscape, false)
		}
	})

	const placementsClasses = {
		'bottom-center': 'origin-top-left left-1/2 transform -translate-x-1/2',
		'bottom-start': 'origin-top-left left-0',
		'bottom-end': 'origin-top-right right-0',
		'top-start': 'origin-bottom-left left-0 bottom-0',
		'top-end': 'origin-bottom-right right-0 bottom-0'
	}
</script>

<div class="relative {wrapperClasses}" bind:this={menu}>
	<!-- svelte-ignore a11y-click-events-have-key-events -->
	<div
		on:click|stopPropagation={() => {
			if (!show) {
				current && current()
				current = close
			}
			show = !show
		}}
		class="relative"
	>
		<slot class="triggerable" name="trigger" />
	</div>
	{#if show}
		<div
			transition:fade|local={{ duration: transitionDuration }}
			class={classNames(
				'z-50 absolute mt-2 rounded-md shadow-lg bg-surface ring-1 ring-black ring-opacity-5 focus:outline-none',
				placementsClasses[placement],
				noMinW ? 'min-w-0' : 'w-60',
				popupClasses
			)}
			role="menu"
			tabindex="-1"
		>
			<slot {close} />
		</div>
	{/if}
</div>
