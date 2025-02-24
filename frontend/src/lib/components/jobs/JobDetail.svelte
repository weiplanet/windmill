<script lang="ts">
	import { goto } from '$app/navigation'
	import { page } from '$app/stores'
	import type { Job } from '$lib/gen'
	import {
		displayDate,
		displayDaysAgo,
		forLater,
		msToSec,
		truncateHash,
		truncateRev
	} from '$lib/utils'
	import {
		faCalendar,
		faCircle,
		faClock,
		faFastForward,
		faHourglassHalf,
		faRobot,
		faSearch,
		faTimes,
		faUser,
		faBarsStaggered
	} from '@fortawesome/free-solid-svg-icons'
	import { CalendarClock } from 'lucide-svelte'
	import { onDestroy, onMount } from 'svelte'
	import Icon from 'svelte-awesome'
	import { check } from 'svelte-awesome/icons'
	import { Badge } from '../common'
	import ScheduleEditor from '../ScheduleEditor.svelte'
	import JobPreview from './JobPreview.svelte'

	const SMALL_ICON_SCALE = 0.7

	export let job: Job
	let scheduleEditor: ScheduleEditor

	let time = Date.now()
	let interval
	onMount(() => {
		interval = setInterval(() => {
			time = Date.now()
		}, 1000)
	})

	onDestroy(() => {
		interval && clearInterval(interval)
	})

	function displayIfRecent(date: Date): string {
		const d = new Date(date)
		const now = new Date()
		const diff = now.getTime() - d.getTime()
		if (diff < 1000 * 600) {
			return `(${displayDaysAgo(d.toString())})`
		} else {
			return ''
		}
	}
	function endedDate(started_at: string, duration_ms: number): string {
		const started = new Date(started_at)
		started.setMilliseconds(started.getMilliseconds() + duration_ms)
		return `${displayDate(started)} ${displayIfRecent(started)}`
	}
</script>

<ScheduleEditor on:update={() => goto('/schedules')} bind:this={scheduleEditor} />

<div class="border rounded py-4">
	<div class="grid grid-cols-1 lg:grid-cols-4 w-full gap-4">
		<div class="flex-col col-span-2">
			<div class="flex flex-row text-sm">
				{#if job === undefined}
					No job found
				{:else}
					<div class="block text-center align-middle px-6">
						{#if 'success' in job && job.success}
							{#if job.is_skipped}
								<Icon
									class="text-green-600 dark:text-green-400"
									data={faFastForward}
									scale={SMALL_ICON_SCALE}
									label="Job completed successfully but was skipped"
								/>
							{:else}
								<Icon
									class="text-green-600 dark:text-green-400"
									data={check}
									scale={SMALL_ICON_SCALE}
									label="Job completed successfully"
								/>
							{/if}
						{:else if 'success' in job}
							<Icon
								class="text-red-600 dark:text-red-400"
								data={faTimes}
								scale={SMALL_ICON_SCALE}
								label="Job completed with an error"
							/>
						{:else if 'running' in job && job.running}
							<Icon
								class="text-yellow-500"
								data={faCircle}
								scale={SMALL_ICON_SCALE}
								label="Job is running"
							/>
						{:else if job && 'running' in job && job.scheduled_for && forLater(job.scheduled_for)}
							<Icon
								class="text-secondary"
								data={faCalendar}
								scale={SMALL_ICON_SCALE}
								label="Job is scheduled to run at a later time"
							/>
						{:else}
							<Icon
								class="text-tertiary"
								data={faHourglassHalf}
								scale={SMALL_ICON_SCALE}
								label="Job is waiting for an executor"
							/>
						{/if}
					</div>

					<div class="flex flex-row space-x-2">
						<JobPreview id={job.id}>
							<div class="whitespace-nowrap">
								{#if job.script_path}
									<a href="/run/{job.id}?workspace={job.workspace_id}">{job.script_path} </a>
								{:else if 'job_kind' in job && job.job_kind == 'preview'}
									<a href="/run/{job.id}?workspace={job.workspace_id}">Preview without path </a>
								{:else if 'job_kind' in job && job.job_kind == 'dependencies'}
									<a href="/run/{job.id}?workspace={job.workspace_id}"
										>lock deps of {truncateHash(job.script_hash ?? '')}</a
									>
								{:else if 'job_kind' in job && job.job_kind == 'identity'}
									<a href="/run/{job.id}?workspace={job.workspace_id}">no op</a>
								{/if}
								<button
									class="ml-1"
									on:click={() => {
										goto(`/runs/${job.script_path}?${$page.url.searchParams.toString()}`)
									}}><Badge><Icon scale={0.7} data={faSearch} /></Badge></button
								>
							</div>
						</JobPreview>
						<div class="whitespace-nowrap">
							{#if 'job_kind' in job}<a href="/run/{job.id}"
									><Badge color="blue">{job.job_kind}</Badge></a
								>
							{/if}
							{#if job.is_flow_step && (job.job_kind == 'script' || job.job_kind == 'preview')}
								<a href="/run/{job.parent_job}?workspace={job.workspace_id}"
									><Badge color="gray">flow step</Badge></a
								>
							{/if}
						</div>
					</div>
				{/if}
			</div>
			<div class="pl-14 italic text-secondary text-2xs whitespace-nowrap overflow-hidden">
				{truncateRev(job.id, 8, '')}
			</div>
		</div>
		<div class="bg-surface grid grid-cols-2 gap-x-2 col-span-2">
			<div class="w-full text-secondary text-xs text-left flex flex-col gap-1 mx-4 overflow-hidden">
				<div>
					<Icon class="text-secondary" data={faUser} scale={SMALL_ICON_SCALE} /><span class="mx-2">
						By {job.created_by}</span
					>
				</div>
				{#if job && 'duration_ms' in job && job.duration_ms != undefined}
					<div>
						<Icon class="text-secondary" data={faHourglassHalf} scale={SMALL_ICON_SCALE} /><span
							class="mx-2"
						>
							Ran in {msToSec(job.duration_ms)}s</span
						>
					</div>
				{/if}
			</div>
			<div class="text-secondary text-xs text-left place-self-start flex flex-col gap-1">
				{#if 'started_at' in job && job.started_at}
					<div>
						<Icon class="text-secondary" data={faClock} scale={SMALL_ICON_SCALE} /><span
							class="mx-1.5"
						>
							<span>
								{#if job?.['duration_ms']}
									Ended {#key time}
										{endedDate(job.started_at, job?.['duration_ms'])}{/key}
								{:else}
									Started {#key time}
										{displayDaysAgo(job.started_at ?? '')}{/key}
								{/if}</span
							>
						</span></div
					>
				{/if}
				{#if 'scheduled_for' in job && !job.running && job.scheduled_for && forLater(job.scheduled_for)}
					<div class="inline-flex gap-1">
						<CalendarClock size={13} class="-ml-0.5" />
						<span>
							<span class="bg-blue-200 text-secondary text-xs rounded px-1">Scheduled</span>
							for {displayDate(job.scheduled_for ?? '')}
						</span>
					</div>
				{:else if 'scheduled_for' in job && !job.running}
					<div>
						<Icon class="text-secondary" data={faClock} scale={SMALL_ICON_SCALE} /><span
							class="mx-2"
						>
							<span class="bg-blue-200 text-secondary text-xs rounded px-1"
								>Waiting for an executor</span
							>
						</span>
					</div>
				{/if}
				<div>
					{#if job && job.parent_job}
						{#if job.is_flow_step}
							<Icon class="text-secondary" data={faBarsStaggered} scale={SMALL_ICON_SCALE} /><span
								class="mx-1"
							>
								Step of flow <a href={`/run/${job.parent_job}`}>{truncateRev(job.parent_job, 6)}</a
								></span
							>
						{:else}
							<Icon class="text-secondary" data={faRobot} scale={SMALL_ICON_SCALE} /><span
								class="mx-1"
							>
								Parent <a href={`/run/${job.parent_job}`}>{job.parent_job}</a></span
							>
						{/if}
					{:else if job && job.schedule_path}
						<Icon class="text-secondary" data={faCalendar} scale={SMALL_ICON_SCALE} />
						<span class="mx-1"
							>Schedule <button
								class="break-words text-blue-400 font-normal truncate text-xs"
								on:click={() =>
									scheduleEditor?.openEdit(job.schedule_path ?? '', job.job_kind == 'flow')}
								>{job.schedule_path}</button
							></span
						>
					{/if}
				</div>
			</div>
		</div>
	</div>
</div>
