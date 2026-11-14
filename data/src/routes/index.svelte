<script>
	import HKInfo from '$lib/components/HKInfo.svelte';
	import { route } from 'sv-router/generated';
	import { systemInfo } from '$lib/stores/system.svelte.js';

	let busy = $state('');

	const lockState = $derived(
		systemInfo.lock_state === 0 ? 'unlock' : systemInfo.lock_state === 1 ? 'lock' : ''
	);

	async function sendLockAction(action) {
		if (busy) return;
		busy = action;
		try {
			const res = await fetch(`/lock_control?action=${action}`, { method: 'POST' });
			if (res.ok) {
				systemInfo.lock_state = action === 'unlock' ? 0 : 1;
			}
		} catch (e) {
			console.error(e);
		} finally {
			busy = '';
		}
	}
</script>

<div class="card bg-base-200 border border-base-300 mb-4">
	<div class="card-body p-4">
		<div class="flex justify-between w-full">
			<h2 class="card-title text-lg">Управление замком</h2>
			<span
				class="text-sm font-bold"
				class:text-success={lockState === 'unlock'}
				class:text-error={lockState === 'lock'}
			>
				{lockState === 'unlock' ? 'Открыт' : lockState === 'lock' ? 'Закрыт' : ''}
			</span>
		</div>

		<div class="flex gap-2 mt-2">
			<button
				class="btn btn-success flex-1"
				disabled={busy !== ''}
				onclick={() => sendLockAction('unlock')}
			>
				{#if busy === 'unlock'}
					<span class="loading loading-spinner"></span>
				{/if}
				{lockState === 'unlock' ? '✓ Открыто' : 'Открыть'}
			</button>

			<button
				class="btn btn-error flex-1"
				disabled={busy !== ''}
				onclick={() => sendLockAction('lock')}
			>
				{#if busy === 'lock'}
					<span class="loading loading-spinner"></span>
				{/if}
				{lockState === 'lock' ? '✓ Закрыто' : 'Закрыть'}
			</button>
		</div>
	</div>
</div>

<HKInfo hkInfo={route.meta.homeData?.hkInfo} error={route.meta.homeData?.error} />
