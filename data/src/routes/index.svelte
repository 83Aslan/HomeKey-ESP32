<script>
	import HKInfo from '$lib/components/HKInfo.svelte';
	import { route } from 'sv-router/generated';

	let busy = $state('');
	let lock = $state('');

	async function act(a) {
		if (busy) return;
		busy = a;
		try {
			const r = await fetch(`/lock_control?action=${a}`, { method: 'POST' });
			if (r.ok) lock = a;
		} finally {
			busy = '';
		}
	}

	$effect(() => {
		const ws = new WebSocket(`${location.protocol === 'https:' ? 'wss:' : 'ws:'}//${location.host}/ws`);
		ws.onmessage = (e) => {
			try {
				const m = JSON.parse(e.data);
				if (m.type === 'metrics' && m.lock_state !== undefined) {
					lock = m.lock_state === 0 ? 'unlock' : 'lock';
				}
			} catch {}
		};
		return () => ws.close();
	});
</script>

<div class="card bg-base-200 border border-base-300 mb-4 p-4">
	<div class="flex justify-between mb-2">
		<span class="font-bold">Замок</span>
		<span>{lock === 'unlock' ? 'Открыт' : lock === 'lock' ? 'Закрыт' : ''}</span>
	</div>
	<div class="flex gap-2">
		<button class="btn btn-success flex-1" disabled={!!busy} onclick={() => act('unlock')}>
			{busy === 'unlock' ? '...' : lock === 'unlock' ? '✓ Открыто' : 'Открыть'}
		</button>
		<button class="btn btn-error flex-1" disabled={!!busy} onclick={() => act('lock')}>
			{busy === 'lock' ? '...' : lock === 'lock' ? '✓ Закрыто' : 'Закрыть'}
		</button>
	</div>
</div>

<HKInfo hkInfo={route.meta.homeData?.hkInfo} error={route.meta.homeData?.error} />
