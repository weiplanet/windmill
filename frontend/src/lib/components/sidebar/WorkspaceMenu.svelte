<script lang="ts">
	import { switchWorkspace, userWorkspaces, workspaceStore } from '$lib/stores'
	import { classNames } from '$lib/utils'
	import { Building } from 'lucide-svelte'

	import Menu from '../common/menu/Menu.svelte'
	import { faCog, faPlus } from '@fortawesome/free-solid-svg-icons'
	import { Icon } from 'svelte-awesome'

	export let isCollapsed: boolean = false
</script>

<Menu placement="bottom-start" let:close>
	<button
		title="Workspace Menu"
		slot="trigger"
		type="button"
		class={classNames(
			'group w-full flex items-center text-white hover:bg-gray-50 hover:text-gray-900 focus:ring-4 focus:outline-none focus:ring-gray-300 px-2 py-2 text-sm font-medium rounded-md h-8 '
		)}
	>
		<div class="center-center mr-2">
			<Building size={16} />
		</div>

		{#if !isCollapsed}
			<span class={classNames('whitespace-pre truncate')}> {$workspaceStore} </span>
		{/if}
	</button>

	<div class="divide-y" role="none">
		<div class="py-1">
			{#each $userWorkspaces as workspace}
				<button
					class="text-xs min-w-0 w-full overflow-hidden flex flex-col py-1.5
						{$workspaceStore === workspace.id
						? 'cursor-default bg-surface-selected'
						: 'cursor-pointer hover:bg-surface-hover'}"
					on:click={() => {
						if ($workspaceStore === workspace.id) {
							return
						}
						switchWorkspace(workspace.id)
						close()
					}}
				>
					<div class="text-primary pl-4 truncate text-left text-[1.2em]">{workspace.name}</div>
					<div class="text-tertiary font-mono pl-4 text-2xs whitespace-nowrap truncate text-left">
						{workspace.id}
					</div>
				</button>
			{/each}
		</div>
		<div class="py-1" role="none">
			<a
				href="/user/create_workspace"
				class="text-primary block px-4 py-2 text-xs hover:bg-surface-hover hover:text-primary"
				role="menuitem"
				tabindex="-1"
			>
				<Icon data={faPlus} class="-mt-0.5 pr-0.5" /> Workspace
			</a>
		</div>
		<div class="py-1" role="none">
			<a
				href="/user/workspaces"
				on:click={() => {
					localStorage.removeItem('workspace')
				}}
				class="text-primary block px-4 py-2 text-xs hover:bg-surface-hover hover:text-primary"
				role="menuitem"
				tabindex="-1"
			>
				All workspaces & invites
			</a>
		</div>
		<div class="py-1" role="none">
			<a
				href="/workspace_settings"
				class="text-secondary block px-4 py-2 text-xs hover:bg-surface-hover hover:text-primary"
				role="menuitem"
				tabindex="-1"
			>
				<Icon class="pr-0.5" data={faCog} /> Workspace Settings
			</a>
		</div>
	</div>
</Menu>
