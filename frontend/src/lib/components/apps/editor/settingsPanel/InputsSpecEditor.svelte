<script lang="ts">
	import { addWhitespaceBeforeCapitals, capitalize, classNames } from '$lib/utils'
	import Tooltip from '$lib/components/Tooltip.svelte'

	import ConnectedInputEditor from './inputEditor/ConnectedInputEditor.svelte'
	import EvalInputEditor from './inputEditor/EvalInputEditor.svelte'
	import RowInputEditor from './inputEditor/RowInputEditor.svelte'
	import StaticInputEditor from './inputEditor/StaticInputEditor.svelte'
	import UploadInputEditor from './inputEditor/UploadInputEditor.svelte'
	import { getContext } from 'svelte'
	import type { AppViewerContext, RichConfiguration } from '../../types'
	import type { InputConnection, InputType, UploadAppInput } from '../../inputType'
	import ToggleButtonGroup from '$lib/components/common/toggleButton-v2/ToggleButtonGroup.svelte'
	import ToggleButton from '$lib/components/common/toggleButton-v2/ToggleButton.svelte'
	import { FunctionSquare, Pen, Plug2, Upload, User } from 'lucide-svelte'
	import { fieldTypeToTsType } from '../../utils'

	export let id: string
	export let componentInput: RichConfiguration
	export let key: string
	export let userInputEnabled: boolean = false
	export let shouldCapitalize: boolean = true
	export let resourceOnly = false
	export let tooltip: string | undefined = undefined
	export let onlyStatic: boolean = false
	export let fieldType: InputType
	export let subFieldType: InputType | undefined
	export let format: string | undefined
	export let selectOptions: string[] | undefined
	export let fileUpload: UploadAppInput['fileUpload'] | undefined = undefined
	export let placeholder: string | undefined
	export let customTitle: string | undefined = undefined
	export let displayType: boolean = false
	export let noVariablePicker: boolean | undefined = undefined

	const { connectingInput, app } = getContext<AppViewerContext>('AppViewerContext')

	function applyConnection(connection: InputConnection) {
		if (componentInput.type === 'connected') {
			componentInput.connection = connection
			$app = $app
		}
	}

	$: if (componentInput == undefined) {
		//@ts-ignore
		componentInput = {
			type: 'static',
			value: undefined
		}
	}
</script>

{#if !(resourceOnly && (fieldType !== 'object' || !format?.startsWith('resource-')))}
	<div
		class={classNames(
			'flex gap-1',
			onlyStatic && fieldType != 'object' ? 'flex-row items-center justify-between' : 'flex-col'
		)}
	>
		<div class="flex justify-between items-end">
			<div class="flex flex-row gap-4 items-center">
				<span class="text-xs font-semibold truncate text-primary">
					{customTitle
						? customTitle
						: shouldCapitalize
						? capitalize(addWhitespaceBeforeCapitals(key))
						: key}
					{#if tooltip}
						<Tooltip light>
							{tooltip}
						</Tooltip>
					{/if}
				</span>
				{#if displayType}
					<div class="text-xs text-tertiary">
						{fieldType === 'array' && subFieldType
							? `${fieldTypeToTsType(subFieldType)}[]`
							: fieldTypeToTsType(fieldType)}
					</div>
				{/if}
			</div>

			<div class={classNames('flex gap-x-2 gap-y-1 flex-wrap justify-end items-center')}>
				{#if !onlyStatic && componentInput?.type}
					<ToggleButtonGroup
						class="h-7"
						bind:selected={componentInput.type}
						on:selected={(e) => {
							if (e.detail == 'connected' && !componentInput['connection']) {
								$connectingInput = {
									opened: true,
									input: undefined,
									hoveredComponent: undefined,
									onConnect: applyConnection
								}
							} else if (
								e.detail == 'eval' &&
								componentInput['value'] != undefined &&
								(componentInput['expr'] == '' || componentInput['expr'] == undefined)
							) {
								componentInput['expr'] = JSON.stringify(componentInput['value'])
							}
						}}
					>
						<ToggleButton value="static" icon={Pen} iconOnly tooltip="Static" />
						{#if userInputEnabled && !format?.startsWith('resource-')}
							<ToggleButton value="user" icon={User} iconOnly tooltip="User Input" />
						{/if}
						{#if fileUpload}
							<ToggleButton value="upload" icon={Upload} iconOnly tooltip="Upload" />
						{/if}
						<ToggleButton value="connected" icon={Plug2} iconOnly tooltip="Connect" />
						<ToggleButton value="eval" icon={FunctionSquare} iconOnly tooltip="Eval" />
					</ToggleButtonGroup>
				{/if}
			</div>
		</div>

		{#if componentInput?.type === 'connected'}
			<ConnectedInputEditor bind:componentInput />
		{:else if componentInput?.type === 'row'}
			<RowInputEditor bind:componentInput />
		{:else if componentInput?.type === 'static'}
			<div
				class={onlyStatic && fieldType != 'object'
					? 'w-2/3 flex justify-end'
					: 'w-full flex flex-row-reverse'}
			>
				<StaticInputEditor
					{fieldType}
					{subFieldType}
					{selectOptions}
					{format}
					{placeholder}
					noVariablePicker={noVariablePicker ?? false}
					bind:componentInput
				/>
			</div>
		{:else if componentInput?.type === 'eval'}
			<EvalInputEditor {id} bind:componentInput />
		{:else if componentInput?.type === 'upload'}
			<UploadInputEditor bind:componentInput {fileUpload} />
		{:else if componentInput?.type === 'user'}
			<span class="text-2xs italic text-tertiary">Field's value is set by the user</span>
		{/if}
	</div>
{/if}
