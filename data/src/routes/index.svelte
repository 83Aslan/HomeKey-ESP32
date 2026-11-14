<script>
	import HKInfo from '$lib/components/HKInfo.svelte';
	import { route } from 'sv-router/generated';

	let busy = $state(false);

	async function sendLockAction(action) {
		busy = true;
		try {
			await fetch(`/lock_control?action=${action}`, { method: 'POST' });
		} catch (e) {
			console.error(e);
		} finally {
			busy = false;
		}
	}
</script>

<div class="card bg-base-200 shadow-sm border border-base-300 mb-4">
	<div class="card-body p-4">
		<h2 class="card-title text-base">Управление замком</h2>
		<div class="grid grid-cols-2 gap-3 mt-2">
			<button
				class="btn btn-success"
				disabled={busy}
				onclick={() => sendLockAction('unlock')}
			>
				Открыть
			</button>
			<button
				class="btn btn-error"
				disabled={busy}
				onclick={() => sendLockAction('lock')}
			>
				Закрыть
			</button>
		</div>
	</div>
</div>

<HKInfo hkInfo={route.meta.homeData?.hkInfo} error={route.meta.homeData?.error} />
