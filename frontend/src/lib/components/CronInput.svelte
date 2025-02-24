<script lang="ts">
	import { ScheduleService } from '$lib/gen'
	import { emptyString, formatCron } from '$lib/utils'
	import Badge from './Badge.svelte'

	// @ts-ignore
	import Multiselect from 'svelte-multiselect'
	import TimezonePicker from 'svelte-timezone-picker'
	import CollapseLink from './CollapseLink.svelte'
	import { Button } from './common'

	export let schedule: string
	// export let offset: number = -60 * Math.floor(new Date().getTimezoneOffset() / 60)
	export let timezone: string // = Intl.DateTimeFormat().resolvedOptions().timeZone
	export let disabled = false
	export let validCRON = true

	let preview: string[] = []
	// If the user has already entered a cron string, switching to the basic tab will override it.
	let executeEvery: 'second' | 'minute' | 'hour' | 'day-month' | 'month' | 'day-week' = 'minute'

	let seconds = 30
	let minutes = 30
	let hours = 1
	const daysOfMonthOptions: number[] = Array.from(Array(31).keys()).map((i) => i + 1)
	let daysOfMonth: number[] = []
	// let lastDayOfMonth = false
	const monthsOfYearOptions: string[] = [
		'January',
		'February',
		'March',
		'April',
		'May',
		'June',
		'July',
		'August',
		'September',
		'October',
		'November',
		'December'
	]
	let monthsOfYear: string[] = []
	const daysOfWeekOptions: string[] = [
		'Sunday',
		'Monday',
		'Tuesday',
		'Wednesday',
		'Thursday',
		'Friday',
		'Saturday'
	]
	let daysOfWeek: string[] = []
	let UTCTime: string = ''

	$: !emptyString(schedule) && handleScheduleInput(schedule, timezone)

	async function handleScheduleInput(input: string, timezone: string): Promise<void> {
		try {
			preview = await ScheduleService.previewSchedule({
				requestBody: { schedule: formatCron(input), timezone }
			})
			validCRON = true
		} catch (err) {
			if (err.status == 400 && err.body.includes('cron')) {
				validCRON = false
			} else {
				validCRON = false
			}
		}
	}

	let nschedule = ''

	$: {
		// CRON string format
		// sec  min   hour      day of month   month     day of week   year
		// 0    30    9,12,15   1,15           May-Aug   Mon,Wed,Fri   2018/2

		let s_daysOfMonth = ''
		// if (lastDayOfMonth) {
		// 	s_daysOfMonth = 'L'
		// } else
		if (daysOfMonth.length > 0) {
			s_daysOfMonth = daysOfMonth.join(',')
		} else {
			s_daysOfMonth = '*'
		}

		let s_months = ''
		if (monthsOfYear.length > 0) {
			s_months = monthsOfYear.map((m) => m.slice(0, 3).toLowerCase()).join(',')
		} else {
			s_months = '*'
		}

		let s_daysOfWeek = ''
		if (daysOfWeek.length > 0) {
			s_daysOfWeek = daysOfWeek.map((d) => d.slice(0, 3).toLowerCase()).join(',')
		} else {
			s_daysOfWeek = '*'
		}

		const s_AtUTCHours = parseInt(UTCTime.split(':')[0]) || '0'
		const s_AtUTCMinutes = parseInt(UTCTime.split(':')[1]) || '0'

		// If using the basic editor, set the cron string based on the selected options
		if (executeEvery === 'second') {
			if (seconds > 0) {
				nschedule = `*/${seconds} * * * *`
			} else {
				nschedule = `* * * * *`
			}
		} else if (executeEvery === 'minute') {
			if (minutes > 0) {
				nschedule = `0 */${minutes} * * *`
			} else {
				nschedule = `* * * * *`
			}
		} else if (executeEvery === 'hour') {
			if (hours > 0) {
				nschedule = `0 0 */${hours} * *`
			} else {
				nschedule = `* * * * *`
			}
		} else if (executeEvery === 'day-month') {
			nschedule = `0 ${s_AtUTCMinutes} ${s_AtUTCHours} ${s_daysOfMonth} *`
		} else if (executeEvery === 'month') {
			nschedule = `0 ${s_AtUTCMinutes} ${s_AtUTCHours} ${s_daysOfMonth} ${s_months}`
		} else if (executeEvery === 'day-week') {
			nschedule = `0 ${s_AtUTCMinutes} ${s_AtUTCHours} * * ${s_daysOfWeek}`
		}
	}

	$: dateFormatter = new Intl.DateTimeFormat('en-GB', {
		weekday: 'short',
		day: '2-digit',
		month: 'short',
		year: 'numeric',
		hour: 'numeric',
		minute: 'numeric',
		second: 'numeric',
		timeZone: timezone,
		timeZoneName: 'short'
	}).format
</script>

<div class="w-full flex space-x-16">
	<div class="w-full flex flex-col space-y-2">
		<div class="w-full flex flex-col gap-1">
			<small class="font-bold">Cron</small>
			<input
				class="inline-block"
				type="text"
				id="cron-schedule"
				name="cron-schedule"
				placeholder="*/30 * * * *"
				bind:value={schedule}
				{disabled}
			/>
			{#if !validCRON}
				<small class="text-red-600"> Invalid cron syntax </small>
			{/if}
		</div>

		<div class="w-full flex flex-col gap-1">
			<small class="font-bold">Timezone</small>

			{#if disabled}
				<div>
					<Badge>{timezone}</Badge>
				</div>
			{:else}
				<TimezonePicker {timezone} on:update={(e) => (timezone = e.detail.timezone)} />
			{/if}
		</div>

		{#if !disabled}
			<div class="w-full">
				<CollapseLink text="Use simplified builder">
					<div class="w-full flex flex-col gap-4 mt-4">
						<div class="w-full flex flex-col gap-1">
							<small class="font-bold">Execute schedule every</small>

							<div class="w-full flex gap-4">
								<div class="w-full flex flex-col gap-1">
									<select
										{disabled}
										name="execute_every"
										id="execute_every"
										bind:value={executeEvery}
									>
										<option value="second">Second(s)</option>
										<option value="minute">Minute(s)</option>
										<option value="hour">Hour(s)</option>
										<option value="day-month">Day of the month</option>
										<option value="month">Month(s)</option>
										<option value="day-week">Day of the week</option>
									</select>
								</div>

								<div class="w-full flex flex-col gap-1 justify-center">
									{#if executeEvery == 'second'}
										<input {disabled} type="number" min="0" max="59" bind:value={seconds} />
										<small>Valid range 0-59</small>
									{:else if executeEvery == 'minute'}
										<input {disabled} type="number" min="0" max="59" bind:value={minutes} />
										<small>Valid range 0-59</small>
									{:else if executeEvery == 'hour'}
										<input {disabled} type="number" min="0" max="23" bind:value={hours} />
										<small>Valid range 0-23</small>
									{:else if executeEvery == 'day-month'}
										<!-- <div class="w-full flex">
									<label for="lastDayOfMonth" class="w-full flex items-center gap-2">
										<div class="flex">
											<input type="checkbox" id="lastDayOfMonth" bind:checked={lastDayOfMonth} />
										</div>
										<small> Last day of the month </small>
									</label>
								</div> -->
									{/if}
								</div>
							</div>
						</div>

						<div class="w-full flex flex-col gap-4">
							{#if executeEvery == 'month'}
								<div class="w-full flex flex-col">
									<Multiselect
										{disabled}
										bind:selected={monthsOfYear}
										options={monthsOfYearOptions}
										selectedOptionsDraggable={false}
										placeholder="Every month"
									/>
								</div>
							{/if}

							{#if executeEvery == 'day-week'}
								<div class="w-full flex flex-col">
									<Multiselect
										{disabled}
										bind:selected={daysOfWeek}
										options={daysOfWeekOptions}
										selectedOptionsDraggable={false}
										placeholder="Every day"
									/>
								</div>
							{/if}

							{#if executeEvery == 'day-month' || executeEvery == 'month'}
								<div class="w-full flex flex-col gap-1">
									{#if executeEvery == 'month'}
										<small class="font-bold">On day of the month</small>
									{/if}
									<div class="w-full flex gap-4">
										<div class="w-full flex">
											<Multiselect
												{disabled}
												bind:selected={daysOfMonth}
												options={daysOfMonthOptions}
												selectedOptionsDraggable={false}
												placeholder="Every day"
											/>
										</div>

										<!-- {#if executeEvery == 'month'}
									<div class="w-full flex">
										<label for="lastDayOfMonth" class="w-full flex items-center gap-2">
											<div class="flex">
												<input type="checkbox" id="lastDayOfMonth" bind:checked={lastDayOfMonth} />
											</div>
											<small> Last day of the month </small>
										</label>
									</div>
								{/if} -->
									</div>
									<small>Schedule will only execute on valid calendar days</small>
								</div>
							{/if}

							{#if executeEvery == 'day-month' || executeEvery == 'month' || executeEvery == 'day-week'}
								<div class="w-full flex flex-col gap-1">
									<small class="font-bold">At UTC Time</small>
									<input
										{disabled}
										type="time"
										name="atUTCTime"
										id="atUTCTime"
										bind:value={UTCTime}
									/>
								</div>
							{/if}
						</div>

						<div class="w-full flex flex-col gap-1">
							<small class="font-bold">Preview New Cron</small>

							<div class="flex p-2 px-4 rounded-md bg-surface-secondary">
								<span>{nschedule}</span>
							</div>
						</div>
					</div>

					<div class="mt-4">
						<Button color="dark" size="xs" on:click={() => (schedule = nschedule)}>
							Set Cron Schedule
						</Button>
					</div>
				</CollapseLink>
			</div>
		{/if}
	</div>

	<div class="w-full flex flex-col space-y-2">
		<h3>Execution summary</h3>
		<hr />
		<div class="flex flex-col space-y-2">
			<small>Estimated upcoming events ({timezone})</small>
			<div class="flex flex-col rounded-md p-4 border text-tertiary bg-surface-secondary">
				{#each preview as date}
					<div class="flex items-center space-x-2 text-sm">
						<span>{dateFormatter(new Date(date))}</span>
					</div>
				{/each}
			</div>
		</div>
	</div>
</div>
