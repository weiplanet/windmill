<script lang="ts">
	import DisplayResult from '$lib/components/DisplayResult.svelte'
	import { getContext } from 'svelte'
	import { twMerge } from 'tailwind-merge'
	import { initOutput } from '../../editor/appUtils'
	import type { AppInput } from '../../inputType'
	import {
		IS_APP_PUBLIC_CONTEXT_KEY,
		type AppViewerContext,
		type ComponentCustomCSS
	} from '../../types'
	import RunnableWrapper from '../helpers/RunnableWrapper.svelte'
	import { concatCustomCss } from '../../utils'

	export let id: string
	export let componentInput: AppInput | undefined
	export let initializing: boolean | undefined = undefined
	export let customCss: ComponentCustomCSS<'displaycomponent'> | undefined = undefined
	export let render: boolean

	const requireHtmlApproval = getContext<boolean | undefined>(IS_APP_PUBLIC_CONTEXT_KEY)
	const { app, worldStore, componentControl } = getContext<AppViewerContext>('AppViewerContext')
	let result: any = undefined

	$componentControl[id] = {
		setValue(value: string) {
			result = value
		}
	}

	const outputs = initOutput($worldStore, id, {
		result: undefined,
		loading: false
	})

	$: css = concatCustomCss($app.css?.displaycomponent, customCss)
</script>

<RunnableWrapper {outputs} {render} {componentInput} {id} bind:initializing bind:result>
	<div class="flex flex-col w-full h-full">
		<div
			class={twMerge(
				'w-full border-b px-2 text-xs p-1 font-semibold bg-gray-500 text-white rounded-t-sm',
				css?.header?.class
			)}
			style={css?.header?.style}
		>
			Results
		</div>
		<div
			style={twMerge(
				$app.css?.['displaycomponent']?.['container']?.style,
				customCss?.container?.style
			)}
			class={twMerge(
				'p-2 grow overflow-auto',
				$app.css?.['displaycomponent']?.['container']?.class,
				customCss?.container?.class
			)}
		>
			<DisplayResult {result} {requireHtmlApproval} />
		</div>
	</div>
</RunnableWrapper>
