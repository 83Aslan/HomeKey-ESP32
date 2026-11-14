<script>
	import HKInfo from '$lib/components/HKInfo.svelte';
	import { route } from 'sv-router/generated';
	import { onMount } from 'svelte';

	let busyAction = $state(null);
	let lockState = $state(null);

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
			<span class="text-sm font-medium">
				{lockState === 'unlock' ? 'Открыт' : lockState === 'lock' ? 'Закрыт' : ''}
			</span>
		</div>

		<div class="flex gap-3 mt-2">
			<button
				class="btn btn-success flex-1"
				disabled={busyAction !== null}
				onclick={() => sendLockAction('unlock')}
			>
				{busyAction === 'unlock' ? '...' : lockState === 'unlock' ? '✓ Открыто' : 'Открыть'}
			</button>

			<button
				class="btn btn-error flex-1"
				disabled={busyAction !== null}
				onclick={() => sendLockAction('lock')}
			>
				{busyAction === 'lock' ? '...' : lockState === 'lock' ? '✓ Закрыто' : 'Закрыть'}
			</button>
		</div>
	</div>
</div>

<HKInfo hkInfo={route.meta.homeData?.hkInfo} error={route.meta.homeData?.error} />
