<script>
	import HKInfo from '$lib/components/HKInfo.svelte';
	import { route } from 'sv-router/generated';
	import { onMount } from 'svelte';

	let busyAction = $state(null);
	let lockState = $state(null); // 'unlock' | 'lock' | null

	async function sendLockAction(action) {
		if (busyAction) return;
		busyAction = action;
		try {
			const res = await fetch(`/lock_control?action=${action}`, { method: 'POST' });
			if (res.ok) {
				lockState = action;
			}
		} catch (e) {
			console.error(e);
		} finally {
			busyAction = null;
		}
	}

	onMount(() => {
		const proto = location.protocol === 'https:' ? 'wss:' : 'ws:';
		const ws = new WebSocket(`${proto}//${location.host}/ws`);
		ws.onmessage = (event) => {
			try {
				const msg = JSON.parse(event.data);
				if (msg.type === 'metrics' && typeof msg.lock_state === 'number') {
					lockState = msg.lock_state === 0 ? 'unlock' : 'lock';
				}
			} catch {}
		};
		return () => ws.close();
	});
</script>

<div class="card bg-base-200 shadow-sm border border-base-300 mb-4">
	<div class="card-body p-4">
		<div class="flex items-center justify-between">
			<h2 class="card-title text-base">Управление замком</h2>
			{#if lockState === 'unlock'}
				<span class="badge badge-success badge-sm">Открыт</span>
			{:else if lockState === 'lock'}
				<span class="badge badge-error badge-sm">Закрыт</span>
			{/if}
		</div>

		<div class="grid grid-cols-2 gap-3 mt-2">
			<button
				class="btn btn-success {lockState && lockState !== 'unlock' ? 'btn-outline' : ''}"
				disabled={busyAction !== null}
				onclick={() => sendLockAction('unlock')}
			>
				{#if busyAction === 'unlock'}
					<span class="loading loading-spinner loading-xs"></span>
				{/if}
				{lockState === 'unlock' ? '✓ Открыто' : 'Открыть'}
			</button>

			<button
				class="btn btn-error {lockState && lockState !== 'lock' ? 'btn-outline' : ''}"
				disabled={busyAction !== null}
				onclick={() => sendLockAction('lock')}
			>
				{#if busyAction === 'lock'}
					<span class="loading loading-spinner loading-xs"></span>
				{/if}
				{lockState === 'lock' ? '✓ Закрыто' : 'Закрыть'}
			</button>
		</div>
	</div>
</div>

<HKInfo hkInfo={route.meta.homeData?.hkInfo} error={route.meta.homeData?.error} />
