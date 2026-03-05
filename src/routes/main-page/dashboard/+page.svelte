<script lang="ts">
	import { onMount } from 'svelte';

	// ── Types ──────────────────────────────────────────────────────────────
	type RekapKamar = {
		gedung: string;
		nomorKamar: string;
		totalPelanggaran: number;
		skorAkhir: number;
	};

	type RekapSantri = {
		nama: string;
		gedung: string;
		nomorKamar: string;
		totalPelanggaran: number;
		skorAkhir: number;
		hpTime: string;
	};

	type RekapByGedung<T> = Record<string, T[]>;

	// ── State ──────────────────────────────────────────────────────────────
	let activeTab: 'kamar' | 'santri' = 'kamar';

	let rekapKamar: RekapByGedung<RekapKamar> = {};
	let rekapSantri: RekapByGedung<RekapSantri> = {};

	let loadingKamar = true;
	let loadingSantri = true;
	let errorKamar = '';
	let errorSantri = '';

	let activeGedungKamar = '';
	let activeGedungSantri = '';

	// ── Derived ────────────────────────────────────────────────────────────
	$: gedungListKamar = Object.keys(rekapKamar);
	$: gedungListSantri = Object.keys(rekapSantri);

	$: if (gedungListKamar.length && !activeGedungKamar) {
		activeGedungKamar = gedungListKamar[0];
	}
	$: if (gedungListSantri.length && !activeGedungSantri) {
		activeGedungSantri = gedungListSantri[0];
	}

	$: currentKamar = rekapKamar[activeGedungKamar] ?? [];
	$: currentSantri = rekapSantri[activeGedungSantri] ?? [];

	$: avgSkorKamar = currentKamar.length
		? Math.round(currentKamar.reduce((a, b) => a + b.skorAkhir, 0) / currentKamar.length)
		: 0;
	$: avgSkorSantri = currentSantri.length
		? Math.round(currentSantri.reduce((a, b) => a + b.skorAkhir, 0) / currentSantri.length)
		: 0;

	// ── Fetch ──────────────────────────────────────────────────────────────
	async function fetchData(): Promise<void> {
		const token = localStorage.getItem('token') ?? '';
		const headers = { Authorization: `Bearer ${token}` };

		// Kamar
		loadingKamar = true;
		try {
			const res = await fetch('http://localhost:3000/api/rekap-kamar-mingguan', { headers });
			if (!res.ok) throw new Error('Gagal mengambil data kamar');
			rekapKamar = await res.json();
		} catch (e) {
			errorKamar = e instanceof Error ? e.message : 'Terjadi kesalahan';
		} finally {
			loadingKamar = false;
		}

		// Santri
		loadingSantri = true;
		try {
			const res = await fetch('http://localhost:3000/api/rekap-mingguan', { headers });
			if (!res.ok) throw new Error('Gagal mengambil data santri');
			rekapSantri = await res.json();
		} catch (e) {
			errorSantri = e instanceof Error ? e.message : 'Terjadi kesalahan';
		} finally {
			loadingSantri = false;
		}
	}

	onMount(fetchData);

	// ── Helpers ────────────────────────────────────────────────────────────
	function getSkorColor(skor: number): string {
		if (skor >= 90) return 'text-emerald-600';
		if (skor >= 80) return 'text-amber-500';
		return 'text-red-500';
	}

	function getSkorBg(skor: number): string {
		if (skor >= 90) return 'bg-emerald-50 border-emerald-100';
		if (skor >= 80) return 'bg-amber-50 border-amber-100';
		return 'bg-red-50 border-red-100';
	}

	function getSkorBar(skor: number): string {
		if (skor >= 90) return 'bg-emerald-500';
		if (skor >= 80) return 'bg-amber-400';
		return 'bg-red-400';
	}

	function getSkorLabel(skor: number): string {
		if (skor >= 90) return 'Baik';
		if (skor >= 80) return 'Cukup';
		return 'Kurang';
	}

	function getHpBadge(hpTime: string): string {
		if (hpTime === '+30 min') return 'bg-emerald-100 text-emerald-700';
		if (hpTime === 'normal') return 'bg-blue-100 text-blue-700';
		return 'bg-gray-100 text-gray-500';
	}

	function getRankBadge(index: number): string {
		if (index === 0) return 'bg-amber-400 text-white';
		if (index === 1) return 'bg-gray-300 text-gray-700';
		if (index === 2) return 'bg-amber-600 text-white';
		return 'bg-gray-100 text-gray-500';
	}
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link
		href="https://fonts.googleapis.com/css2?family=DM+Sans:opsz,wght@9..40,400;9..40,500;9..40,600;9..40,700&family=DM+Mono:wght@500&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div
	class="relative min-h-screen overflow-x-hidden"
	style="background: #f8faf9; background-image: radial-gradient(circle at 10% 0%, rgba(110,231,183,0.1) 0%, transparent 40%);"
>
	<!-- Page header -->
	<div class="mx-auto max-w-3xl px-4 pt-6 pb-2">
		<div class="flex items-center justify-between gap-4">
			<div>
				<h1 class="text-xl leading-none font-bold text-gray-900">Rekap Mingguan</h1>
				<p class="mt-1 text-xs text-gray-400">7 hari terakhir · Diperbarui otomatis</p>
			</div>
			<button
				type="button"
				on:click={fetchData}
				class="flex items-center gap-1.5 rounded-xl border border-gray-200 bg-white px-3 py-2 text-xs font-semibold text-gray-600 shadow-sm transition-all hover:border-gray-300 hover:bg-gray-50"
				aria-label="Muat ulang data"
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-3.5 w-3.5"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
					stroke-width="2.5"
					aria-hidden="true"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M4 4v5h5M20 20v-5h-5M4.34 15a8 8 0 1015.32-6"
					/>
				</svg>
				Refresh
			</button>
		</div>
	</div>

	<main class="mx-auto max-w-3xl space-y-4 px-4 py-4 pb-10">
		<!-- Tab switcher -->
		<div
			class="flex gap-1 rounded-2xl border border-gray-100 bg-white p-1.5 shadow-sm"
			role="tablist"
		>
			<button
				type="button"
				role="tab"
				aria-selected={activeTab === 'kamar'}
				class="flex flex-1 items-center justify-center gap-2 rounded-xl py-2.5 text-sm font-semibold transition-all duration-200
          {activeTab === 'kamar'
					? 'bg-linear-to-r from-emerald-500 to-teal-600 text-white shadow-md shadow-emerald-200'
					: 'text-gray-400 hover:text-gray-600'}"
				on:click={() => (activeTab = 'kamar')}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-4 w-4"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
					stroke-width="2"
					aria-hidden="true"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"
					/>
				</svg>
				Rekap Kamar
			</button>
			<button
				type="button"
				role="tab"
				aria-selected={activeTab === 'santri'}
				class="flex flex-1 items-center justify-center gap-2 rounded-xl py-2.5 text-sm font-semibold transition-all duration-200
          {activeTab === 'santri'
					? 'bg-linear-to-r from-emerald-500 to-teal-600 text-white shadow-md shadow-emerald-200'
					: 'text-gray-400 hover:text-gray-600'}"
				on:click={() => (activeTab = 'santri')}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-4 w-4"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
					stroke-width="2"
					aria-hidden="true"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"
					/>
				</svg>
				Rekap Santri
			</button>
		</div>

		<!-- ══════════ KAMAR TAB ══════════ -->
		{#if activeTab === 'kamar'}
			<div class="fade-up space-y-4">
				{#if loadingKamar}
					<!-- Skeleton -->
					<div class="space-y-3">
						{#each Array(4) as _}
							<div class="space-y-3 rounded-2xl border border-gray-100 bg-white p-4">
								<div class="skeleton h-4 w-32"></div>
								<div class="skeleton h-3 w-full"></div>
								<div class="skeleton h-3 w-3/4"></div>
							</div>
						{/each}
					</div>
				{:else if errorKamar}
					<div
						class="flex items-center gap-2 rounded-2xl border border-red-100 bg-red-50 p-4 text-sm text-red-600"
						role="alert"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-4 w-4 shrink-0"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
							stroke-width="2"
							aria-hidden="true"
						>
							<circle cx="12" cy="12" r="10" /><line x1="12" y1="8" x2="12" y2="12" /><line
								x1="12"
								y1="16"
								x2="12.01"
								y2="16"
							/>
						</svg>
						{errorKamar}
					</div>
				{:else if gedungListKamar.length === 0}
					<div class="rounded-2xl border border-gray-100 bg-white p-8 text-center">
						<p class="text-sm text-gray-400">Belum ada data penilaian kamar</p>
					</div>
				{:else}
					<!-- Gedung filter chips -->
					<div class="scrollbar-hide flex gap-2 overflow-x-auto pb-1">
						{#each gedungListKamar as gedung}
							<button
								type="button"
								class="gedung-chip {activeGedungKamar === gedung ? 'active' : 'inactive'}"
								on:click={() => (activeGedungKamar = gedung)}
							>
								Gedung {gedung}
							</button>
						{/each}
					</div>

					<!-- Stat summary -->
					<div class="grid grid-cols-3 gap-2">
						<div class="stat-card">
							<p class="text-xs font-medium text-gray-400">Total Kamar</p>
							<p class="text-xl font-bold text-gray-800" style="font-family:'DM Mono',monospace">
								{currentKamar.length}
							</p>
						</div>
						<div class="stat-card">
							<p class="text-xs font-medium text-gray-400">Rata-rata Skor</p>
							<p
								class="text-xl font-bold {getSkorColor(avgSkorKamar)}"
								style="font-family:'DM Mono',monospace"
							>
								{avgSkorKamar}
							</p>
						</div>
						<div class="stat-card">
							<p class="text-xs font-medium text-gray-400">Kamar Baik</p>
							<p class="text-xl font-bold text-emerald-600" style="font-family:'DM Mono',monospace">
								{currentKamar.filter((k) => k.skorAkhir >= 90).length}
							</p>
						</div>
					</div>

					<!-- Kamar list -->
					<div class="overflow-hidden rounded-3xl border border-gray-100 bg-white shadow-sm">
						<div class="flex items-center justify-between border-b border-gray-50 px-5 py-4">
							<p class="text-sm font-bold text-gray-800">Gedung {activeGedungKamar}</p>
							<span class="rounded-full bg-gray-100 px-2.5 py-1 text-xs font-medium text-gray-400">
								{currentKamar.length} kamar
							</span>
						</div>

						<div class="divide-y divide-gray-50">
							{#each currentKamar as kamar, i}
								<div
									class="flex items-center gap-3 px-5 py-3.5 transition-colors hover:bg-gray-50/50"
								>
									<!-- Rank -->
									<span class="rank-num {getRankBadge(i)}">{i + 1}</span>

									<!-- Nomor kamar -->
									<div class="min-w-0 flex-1">
										<div class="mb-1 flex items-center gap-2">
											<p class="text-sm font-semibold text-gray-800">Kamar {kamar.nomorKamar}</p>
											<span
												class="rounded-full border px-2 py-0.5 text-xs font-semibold {getSkorBg(
													kamar.skorAkhir
												)} {getSkorColor(kamar.skorAkhir)}"
											>
												{getSkorLabel(kamar.skorAkhir)}
											</span>
										</div>
										<!-- Score bar -->
										<div class="score-bar-track w-full">
											<div
												class="score-bar-fill {getSkorBar(kamar.skorAkhir)}"
												style="width: {kamar.skorAkhir}%"
												role="progressbar"
												aria-valuenow={kamar.skorAkhir}
												aria-valuemin={0}
												aria-valuemax={95}
												aria-label="Skor kamar {kamar.nomorKamar}"
											></div>
										</div>
										<p class="mt-1 text-xs text-gray-400">{kamar.totalPelanggaran} pelanggaran</p>
									</div>

									<!-- Skor -->
									<div class="shrink-0 text-right">
										<p
											class="text-lg font-bold {getSkorColor(kamar.skorAkhir)}"
											style="font-family:'DM Mono',monospace"
										>
											{kamar.skorAkhir}
										</p>
										<p class="text-xs text-gray-400">/ 95</p>
									</div>
								</div>
							{/each}
						</div>
					</div>
				{/if}
			</div>
		{/if}

		<!-- ══════════ SANTRI TAB ══════════ -->
		{#if activeTab === 'santri'}
			<div class="fade-up space-y-4">
				{#if loadingSantri}
					<div class="space-y-3">
						{#each Array(5) as _}
							<div class="space-y-3 rounded-2xl border border-gray-100 bg-white p-4">
								<div class="skeleton h-4 w-40"></div>
								<div class="skeleton h-3 w-full"></div>
								<div class="skeleton h-3 w-2/3"></div>
							</div>
						{/each}
					</div>
				{:else if errorSantri}
					<div
						class="flex items-center gap-2 rounded-2xl border border-red-100 bg-red-50 p-4 text-sm text-red-600"
						role="alert"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-4 w-4 shrink-0"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
							stroke-width="2"
							aria-hidden="true"
						>
							<circle cx="12" cy="12" r="10" /><line x1="12" y1="8" x2="12" y2="12" /><line
								x1="12"
								y1="16"
								x2="12.01"
								y2="16"
							/>
						</svg>
						{errorSantri}
					</div>
				{:else if gedungListSantri.length === 0}
					<div class="rounded-2xl border border-gray-100 bg-white p-8 text-center">
						<p class="text-sm text-gray-400">Belum ada data penilaian santri</p>
					</div>
				{:else}
					<!-- Gedung filter chips -->
					<div class="flex gap-2 overflow-x-auto pb-1">
						{#each gedungListSantri as gedung}
							<button
								type="button"
								class="gedung-chip {activeGedungSantri === gedung ? 'active' : 'inactive'}"
								on:click={() => (activeGedungSantri = gedung)}
							>
								Gedung {gedung}
							</button>
						{/each}
					</div>

					<!-- Stat summary -->
					<div class="grid grid-cols-3 gap-2">
						<div class="stat-card">
							<p class="text-xs font-medium text-gray-400">Total Santri</p>
							<p class="text-xl font-bold text-gray-800" style="font-family:'DM Mono',monospace">
								{currentSantri.length}
							</p>
						</div>
						<div class="stat-card">
							<p class="text-xs font-medium text-gray-400">Rata-rata Skor</p>
							<p
								class="text-xl font-bold {getSkorColor(avgSkorSantri)}"
								style="font-family:'DM Mono',monospace"
							>
								{avgSkorSantri}
							</p>
						</div>
						<div class="stat-card">
							<p class="text-xs font-medium text-gray-400">+30 Menit HP</p>
							<p class="text-xl font-bold text-emerald-600" style="font-family:'DM Mono',monospace">
								{currentSantri.filter((s) => s.hpTime === '+30 min').length}
							</p>
						</div>
					</div>

					<!-- HP time legend -->
					<div
						class="flex items-center gap-3 rounded-2xl border border-gray-100 bg-white px-4 py-2.5 text-xs text-gray-500"
					>
						<span class="font-semibold text-gray-600">Durasi HP:</span>
						<span class="flex items-center gap-1">
							<span class="inline-block h-2 w-2 rounded-full bg-emerald-400"></span>
							Skor 90–95 = +30 menit
						</span>
						<span class="flex items-center gap-1">
							<span class="inline-block h-2 w-2 rounded-full bg-blue-400"></span>
							Skor 80–89 = Normal
						</span>
					</div>

					<!-- Santri list -->
					<div class="overflow-hidden rounded-3xl border border-gray-100 bg-white shadow-sm">
						<div class="flex items-center justify-between border-b border-gray-50 px-5 py-4">
							<p class="text-sm font-bold text-gray-800">Gedung {activeGedungSantri}</p>
							<span class="rounded-full bg-gray-100 px-2.5 py-1 text-xs font-medium text-gray-400">
								{currentSantri.length} santri
							</span>
						</div>

						<div class="divide-y divide-gray-50">
							{#each currentSantri as santri, i}
								<div
									class="flex items-center gap-3 px-5 py-3.5 transition-colors hover:bg-gray-50/50"
								>
									<!-- Rank -->
									<span class="rank-num {getRankBadge(i)}">{i + 1}</span>

									<!-- Avatar -->
									<div
										class="flex h-8 w-8 shrink-0 items-center justify-center rounded-full bg-linear-to-br from-emerald-400 to-teal-500 text-xs font-bold text-white"
										aria-hidden="true"
									>
										{santri.nama.charAt(0).toUpperCase()}
									</div>

									<!-- Info -->
									<div class="min-w-0 flex-1">
										<div class="mb-1 flex flex-wrap items-center gap-2">
											<p class="truncate text-sm font-semibold text-gray-800">{santri.nama}</p>
											{#if santri.hpTime}
												<span
													class="rounded-full px-2 py-0.5 text-xs font-semibold {getHpBadge(
														santri.hpTime
													)}"
												>
													📱 {santri.hpTime}
												</span>
											{/if}
										</div>
										<div class="score-bar-track w-full">
											<div
												class="score-bar-fill {getSkorBar(santri.skorAkhir)}"
												style="width: {santri.skorAkhir}%"
												role="progressbar"
												aria-valuenow={santri.skorAkhir}
												aria-valuemin={0}
												aria-valuemax={95}
												aria-label="Skor {santri.nama}"
											></div>
										</div>
										<p class="mt-1 text-xs text-gray-400">
											Kamar {santri.nomorKamar} · {santri.totalPelanggaran} pelanggaran
										</p>
									</div>

									<!-- Skor -->
									<div class="shrink-0 text-right">
										<p
											class="text-lg font-bold {getSkorColor(santri.skorAkhir)}"
											style="font-family:'DM Mono',monospace"
										>
											{santri.skorAkhir}
										</p>
										<p class="text-xs text-gray-400">/ 95</p>
									</div>
								</div>
							{/each}
						</div>
					</div>
				{/if}
			</div>
		{/if}

		<!-- Score legend -->
		<div class="flex flex-wrap items-center justify-center gap-5 pt-1 text-xs text-gray-400">
			<span class="flex items-center gap-1.5">
				<span class="h-2.5 w-2.5 shrink-0 rounded-full bg-emerald-500"></span>
				≥ 90 · Baik
			</span>
			<span class="flex items-center gap-1.5">
				<span class="h-2.5 w-2.5 shrink-0 rounded-full bg-amber-400"></span>
				80–89 · Cukup
			</span>
			<span class="flex items-center gap-1.5">
				<span class="h-2.5 w-2.5 shrink-0 rounded-full bg-red-400"></span>
				&lt; 80 · Kurang
			</span>
		</div>
	</main>
</div>

<style>
	:global(body) {
		font-family: 'DM Sans', sans-serif;
	}

	.score-bar-track {
		height: 5px;
		background: #e5e7eb;
		border-radius: 999px;
		overflow: hidden;
	}
	.score-bar-fill {
		height: 100%;
		border-radius: 999px;
		transition: width 0.6s cubic-bezier(0.4, 0, 0.2, 1);
	}

	.rank-num {
		font-family: 'DM Mono', monospace;
		font-size: 0.7rem;
		font-weight: 500;
		width: 1.5rem;
		height: 1.5rem;
		border-radius: 999px;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
	}

	.gedung-chip {
		padding: 0.35rem 0.85rem;
		border-radius: 999px;
		font-size: 0.78rem;
		font-weight: 600;
		cursor: pointer;
		border: 1.5px solid transparent;
		transition: all 0.15s;
		white-space: nowrap;
	}
	.gedung-chip.active {
		background: white;
		color: #065f46;
		border-color: #a7f3d0;
		box-shadow: 0 1px 4px rgba(0, 0, 0, 0.08);
	}
	.gedung-chip.inactive {
		background: transparent;
		color: #6b7280;
		border-color: #e5e7eb;
	}
	.gedung-chip.inactive:hover {
		border-color: #d1fae5;
		color: #374151;
	}

	@keyframes fadeUp {
		from {
			opacity: 0;
			transform: translateY(8px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
	.fade-up {
		animation: fadeUp 0.25s ease both;
	}

	.stat-card {
		background: white;
		border-radius: 1.25rem;
		border: 1.5px solid #f0fdf4;
		padding: 1rem 1.25rem;
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
	}

	.skeleton {
		background: linear-gradient(90deg, #f3f4f6 25%, #e5e7eb 50%, #f3f4f6 75%);
		background-size: 200% 100%;
		animation: shimmer 1.4s infinite;
		border-radius: 0.5rem;
	}
	@keyframes shimmer {
		0% {
			background-position: 200% 0;
		}
		100% {
			background-position: -200% 0;
		}
	}
</style>
