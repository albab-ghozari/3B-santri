<script lang="ts">
	import { onMount } from 'svelte';
	const baseUrl = import.meta.env.VITE_API_BASE_URL

	type Santri = { id: number; nama: string; kamar?: { nomor: string; gedung?: string } };
	type Kamar = { id: number; nomor: string; gedung: string };
	type KamarKey = 'lantai' | 'kamarMandi' | 'tempatSampah' | 'gantunganBaju';

	let activeTab: 'santri' | 'kamar' = 'santri';

	let allSantri: Santri[] = [];
	let allKamar: Kamar[] = [];
	let loadingSantri = true;
	let loadingKamar = true;

	let santriQuery = '';
	let kamarQuery = '';
	let showSantriDropdown = false;
	let showKamarDropdown = false;

	let selectedSantri: Santri | null = null;
	let selectedKamar: Kamar | null = null;

	// 0 = rapi (default), 1 = tidak rapi (kena poin)
	let nilaiSantri = { rapiTempatTidur: 0, rapiLemari: 0 };
	let nilaiKamar: Record<KamarKey, number> = {
		lantai: 0,
		kamarMandi: 0,
		tempatSampah: 0,
		gantunganBaju: 0
	};

	let isSubmitting = false;
	let alertState: { type: 'success' | 'error'; msg: string } | null = null;

	$: filteredSantri = santriQuery
		? allSantri.filter((s) => s.nama.toLowerCase().includes(santriQuery.toLowerCase())).slice(0, 8)
		: allSantri.slice(0, 8);

	$: filteredKamar = kamarQuery
		? allKamar.filter((k) => k.nomor.toLowerCase().includes(kamarQuery.toLowerCase())).slice(0, 8)
		: allKamar.slice(0, 8);

	onMount(async () => {
		const token = localStorage.getItem('token') ?? '';
		const h = { Authorization: `Bearer ${token}` };
		try {
			const [rS, rK] = await Promise.all([
				fetch(`${baseUrl}/api/santri`, { headers: h }),
				fetch(`${baseUrl}/api/kamar`, { headers: h })
			]);
			allSantri = await rS.json();
			allKamar = await rK.json();
		} catch {
			/* handled by empty state */
		}
		loadingSantri = false;
		loadingKamar = false;
	});

	function showAlert(type: 'success' | 'error', msg: string): void {
		alertState = { type, msg };
		setTimeout(() => (alertState = null), 4000);
	}

	function selectSantri(s: Santri): void {
		selectedSantri = s;
		santriQuery = s.nama;
		showSantriDropdown = false;
	}

	function selectKamar(k: Kamar): void {
		selectedKamar = k;
		kamarQuery = `Kamar ${k.nomor} ${k.gedung ?? ""}`.trim();;
		showKamarDropdown = false;
	}

	function resetSantri(): void {
		selectedSantri = null;
		santriQuery = '';
		nilaiSantri = { rapiTempatTidur: 0, rapiLemari: 0 };
	}

	function resetKamar(): void {
		selectedKamar = null;
		kamarQuery = '';
		nilaiKamar = { lantai: 0, kamarMandi: 0, tempatSampah: 0, gantunganBaju: 0 };
	}

	function toggleKamarNilai(key: KamarKey, checked: boolean): void {
		nilaiKamar = { ...nilaiKamar, [key]: checked ? 1 : 0 };
	}

	async function submitSantri(): Promise<void> {
		if (!selectedSantri) return showAlert('error', 'Pilih santri terlebih dahulu.');
		isSubmitting = true;
		try {
			const res = await fetch(`${baseUrl}/api/nilai-santri`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Bearer ${localStorage.getItem('token')}`
				},
				body: JSON.stringify({ santriId: selectedSantri.id, ...nilaiSantri })
			});
			const data = await res.json();
			if (!res.ok) throw new Error(data.error);
			showAlert('success', `Penilaian ${selectedSantri.nama} berhasil disimpan.`);
			resetSantri();
		} catch (e) {
			showAlert('error', e instanceof Error ? e.message : 'Gagal menyimpan.');
		} finally {
			isSubmitting = false;
		}
	}

	async function submitKamar(): Promise<void> {
		if (!selectedKamar) return showAlert('error', 'Pilih kamar terlebih dahulu.');
		isSubmitting = true;
		try {
			const res = await fetch(`${baseUrl}/api/nilai-kamar`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Bearer ${localStorage.getItem('token')}`
				},
				body: JSON.stringify({ kamarId: selectedKamar.id, ...nilaiKamar })
			});
			const data = await res.json();
			if (!res.ok) throw new Error(data.error);
			showAlert('success', `Penilaian Kamar ${selectedKamar.nomor} berhasil disimpan.`);
			resetKamar();
		} catch (e) {
			showAlert('error', e instanceof Error ? e.message : 'Gagal menyimpan.');
		} finally {
			isSubmitting = false;
		}
	}

	function closeDropdowns(): void {
		showSantriDropdown = false;
		showKamarDropdown = false;
	}

	function stopAndClose(e: Event): void {
		e.stopPropagation();
	}

	const kamarItems: { key: KamarKey; label: string; sub: string }[] = [
		{ key: 'lantai', label: 'Lantai', sub: 'Kebersihan lantai kamar' },
		{ key: 'kamarMandi', label: 'Kamar Mandi', sub: 'Kebersihan kamar mandi' },
		{ key: 'tempatSampah', label: 'Tempat Sampah', sub: 'Kondisi tempat sampah' },
		{ key: 'gantunganBaju', label: 'Gantungan Baju', sub: 'Kerapihan gantungan baju' }
	];
</script>

<svelte:window on:click={closeDropdowns} />

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link
		href="https://fonts.googleapis.com/css2?family=DM+Sans:opsz,wght@9..40,400;9..40,500;9..40,600;9..40,700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div
	class="relative min-h-screen overflow-x-hidden bg-gray-50"
	style="background-image: radial-gradient(circle at 15% 15%, rgba(110,231,183,0.12) 0%, transparent 45%), radial-gradient(circle at 85% 85%, rgba(94,234,212,0.1) 0%, transparent 45%);"
>
	<!-- Header -->
	<header class="sticky top-0 z-40 border-b border-gray-100 bg-white/80 backdrop-blur">
		<div class="mx-auto flex max-w-xl items-center gap-3 px-4 py-3">
			<div
				class="flex h-9 w-9 shrink-0 items-center justify-center rounded-xl bg-linear-to-br from-emerald-500 to-teal-600 shadow-md shadow-emerald-200"
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-4 w-4 text-white"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
					stroke-width="2"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
					/>
				</svg>
			</div>
			<div>
				<p class="text-sm leading-none font-bold text-gray-800">Input Penilaian</p>
				<p class="mt-0.5 text-xs text-gray-400">Sistem Manajemen Pesantren</p>
			</div>
		</div>
	</header>

	<main class="mx-auto max-w-xl space-y-4 px-4 py-5 pb-10">
		<!-- Alert -->
		{#if alertState}
			<div
				class="alert-in flex items-start gap-2.5 rounded-2xl border px-4 py-3 text-sm font-medium
          {alertState.type === 'success'
					? 'border-emerald-100 bg-emerald-50 text-emerald-700'
					: 'border-red-100 bg-red-50 text-red-600'}"
				role="alert"
			>
				{#if alertState.type === 'success'}
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="mt-0.5 h-4 w-4 shrink-0"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
						stroke-width="2.5"
						aria-hidden="true"
					>
						<path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
					</svg>
				{:else}
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="mt-0.5 h-4 w-4 shrink-0"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
						stroke-width="2.5"
						aria-hidden="true"
					>
						<circle cx="12" cy="12" r="10" /><line x1="12" y1="8" x2="12" y2="12" /><line
							x1="12"
							y1="16"
							x2="12.01"
							y2="16"
						/>
					</svg>
				{/if}
				{alertState.msg}
			</div>
		{/if}

		<!-- Tab -->
		<div
			class="flex gap-1 rounded-2xl border border-gray-100 bg-white p-1.5 shadow-sm"
			role="tablist"
		>
			<button
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
				Santri
			</button>
			<button
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
				Kamar
			</button>
		</div>

		<!-- ══ SANTRI FORM ══ -->
		{#if activeTab === 'santri'}
			<div class="fade-up overflow-visible rounded-3xl border border-gray-100 bg-white shadow-sm">
				<div class="border-b border-gray-50 px-5 pt-5 pb-4">
					<p class="font-bold text-gray-800">Penilaian Santri</p>
					<p class="mt-0.5 text-xs text-gray-400">
						Cari santri, lalu centang item yang <span class="font-semibold text-red-400"
							>tidak rapi</span
						>
					</p>
				</div>

				<div class="space-y-3 px-5 py-4">
					{#if !selectedSantri && !selectedKamar }
						<!-- Search santri -->
						<!-- svelte-ignore a11y_no_static_element_interactions -->
						<div
							class="relative"
							on:click={stopAndClose}
							on:keydown={stopAndClose}
							role="button"
							tabindex="0"
						>
							<span
								class="pointer-events-none absolute top-1/2 left-3.5 -translate-y-1/2 text-gray-400"
								aria-hidden="true"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									class="h-4 w-4"
									fill="none"
									viewBox="0 0 24 24"
									stroke="currentColor"
									stroke-width="2"
								>
									<circle cx="11" cy="11" r="8" /><path
										stroke-linecap="round"
										d="M21 21l-4.35-4.35"
									/>
								</svg>
							</span>
							<input
              class="w-full rounded-xl border border-gray-200 bg-gray-50 py-3 pr-4 pl-10 text-sm text-gray-800 placeholder-gray-300 transition-all focus:border-transparent focus:bg-white focus:ring-2 focus:ring-emerald-400 focus:outline-none"
              type="text"
              placeholder="Cari nama santri…"
              bind:value={santriQuery}
              on:focus={() => (showSantriDropdown = true)}
              on:input={() => (showSantriDropdown = true)}
              autocomplete="off"
              aria-label="Cari santri"
              aria-haspopup="listbox"
							/>
							{#if showSantriDropdown}
								<div class="dropdown" role="listbox" aria-label="Daftar santri">
									{#if loadingSantri}
										<div class="px-4 py-3 text-center text-xs text-gray-400" role="status">
											Memuat data santri…
										</div>
									{:else if filteredSantri.length === 0}
										<div class="px-4 py-3 text-center text-xs text-gray-400">
											Santri tidak ditemukan
										</div>
									{:else}
										{#each filteredSantri as s}
											<button
												type="button"
												role="option"
												aria-selected="false"
												class="flex w-full items-center gap-3 px-4 py-2.5 text-left transition-colors hover:bg-emerald-50"
												on:click={() => selectSantri(s)}
											>
												<div
													class="flex h-8 w-8 shrink-0 items-center justify-center rounded-full bg-linear-to-br from-emerald-400 to-teal-500 text-xs font-bold text-white"
													aria-hidden="true"
												>
													{s.nama.charAt(0).toUpperCase()}
												</div>
												<div class="min-w-0">
													<p class="truncate text-sm font-semibold text-gray-800">{s.nama}</p>
													{#if s.kamar}
														<p class="text-xs text-gray-400">Kamar {s.kamar.nomor} {s.kamar?.gedung}</p>
													{/if}
												</div>
											</button>
										{/each}
									{/if}
								</div>
							{/if}
						</div>
					{:else}
						<!-- Selected santri chip -->
						<div
							class="flex items-center gap-3 rounded-xl border border-emerald-100 bg-emerald-50 px-4 py-3"
						>
							<div
								class="flex h-8 w-8 shrink-0 items-center justify-center rounded-full bg-linear-to-br from-emerald-400 to-teal-500 text-xs font-bold text-white"
								aria-hidden="true"
							>
								{selectedSantri?.nama.charAt(0).toUpperCase()}
							</div>
							<div class="min-w-0 flex-1">
								<p class="truncate text-sm font-bold text-emerald-800">{selectedSantri?.nama}</p>
								{#if selectedSantri?.kamar}
									<p class="text-xs text-emerald-500">Kamar {selectedSantri?.kamar.nomor}</p>
								{/if}
							</div>
							<button
								type="button"
								on:click={resetSantri}
								aria-label="Ganti santri"
								class="rounded-lg p-1 text-emerald-300 transition-colors hover:text-emerald-500"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									class="h-4 w-4"
									fill="none"
									viewBox="0 0 24 24"
									stroke="currentColor"
									stroke-width="2.5"
									aria-hidden="true"
								>
									<path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
								</svg>
							</button>
						</div>
					{/if}

					<!-- Checklist — muncul setelah santri dipilih -->
					{#if selectedSantri}
						<div class="space-y-2 pt-1">
							<div class="flex items-center justify-between">
								<p class="text-xs font-semibold tracking-widest text-gray-500 uppercase">
									Item Penilaian
								</p>
								<span class="rounded-full bg-gray-100 px-2 py-0.5 text-xs text-gray-400">
									{nilaiSantri.rapiTempatTidur + nilaiSantri.rapiLemari} poin
								</span>
							</div>

							<!-- Tempat Tidur -->
							<label class="check-row {nilaiSantri.rapiTempatTidur === 1 ? 'is-checked' : ''}">
								<input
									type="checkbox"
									class="sr-only"
									checked={nilaiSantri.rapiTempatTidur === 1}
									on:change={(e) =>
										(nilaiSantri = {
											...nilaiSantri,
											rapiTempatTidur: e.currentTarget.checked ? 1 : 0
										})}
									aria-label="Tempat tidur tidak rapi"
								/>
								<div class="flex items-center gap-3">
									<span
										class="shrink-0 {nilaiSantri.rapiTempatTidur === 1
											? 'text-red-400'
											: 'text-gray-400'}"
										aria-hidden="true"
									>
										<svg
											xmlns="http://www.w3.org/2000/svg"
											class="h-5 w-5"
											fill="none"
											viewBox="0 0 24 24"
											stroke="currentColor"
											stroke-width="1.8"
										>
											<path
												stroke-linecap="round"
												stroke-linejoin="round"
												d="M3 12h18M3 12V8a2 2 0 012-2h14a2 2 0 012 2v4M3 12v5a1 1 0 001 1h16a1 1 0 001-1v-5"
											/>
										</svg>
									</span>
									<div>
										<p
											class="text-sm font-semibold {nilaiSantri.rapiTempatTidur === 1
												? 'text-red-700'
												: 'text-gray-700'}"
										>
											Tempat Tidur
										</p>
										<p
											class="text-xs {nilaiSantri.rapiTempatTidur === 1
												? 'text-red-400'
												: 'text-gray-400'}"
										>
											{nilaiSantri.rapiTempatTidur === 1
												? '⚠ Tidak rapi · +1 poin'
												: 'Rapi · 0 poin'}
										</p>
									</div>
								</div>
								<div
									class="cb-box {nilaiSantri.rapiTempatTidur === 1 ? 'is-checked' : ''}"
									aria-hidden="true"
								>
									{#if nilaiSantri.rapiTempatTidur === 1}
										<svg
											xmlns="http://www.w3.org/2000/svg"
											class="h-3 w-3 text-white"
											fill="none"
											viewBox="0 0 24 24"
											stroke="currentColor"
											stroke-width="3.5"
										>
											<path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
										</svg>
									{/if}
								</div>
							</label>

							<!-- Lemari -->
							<label class="check-row {nilaiSantri.rapiLemari === 1 ? 'is-checked' : ''}">
								<input
									type="checkbox"
									class="sr-only"
									checked={nilaiSantri.rapiLemari === 1}
									on:change={(e) =>
										(nilaiSantri = { ...nilaiSantri, rapiLemari: e.currentTarget.checked ? 1 : 0 })}
									aria-label="Lemari tidak rapi"
								/>
								<div class="flex items-center gap-3">
									<span
										class="shrink-0 {nilaiSantri.rapiLemari === 1
											? 'text-red-400'
											: 'text-gray-400'}"
										aria-hidden="true"
									>
										<svg
											xmlns="http://www.w3.org/2000/svg"
											class="h-5 w-5"
											fill="none"
											viewBox="0 0 24 24"
											stroke="currentColor"
											stroke-width="1.8"
										>
											<rect x="3" y="3" width="18" height="18" rx="2" />
											<line x1="12" y1="3" x2="12" y2="21" />
											<circle cx="7.5" cy="12" r="1" />
											<circle cx="16.5" cy="12" r="1" />
										</svg>
									</span>
									<div>
										<p
											class="text-sm font-semibold {nilaiSantri.rapiLemari === 1
												? 'text-red-700'
												: 'text-gray-700'}"
										>
											Lemari
										</p>
										<p
											class="text-xs {nilaiSantri.rapiLemari === 1
												? 'text-red-400'
												: 'text-gray-400'}"
										>
											{nilaiSantri.rapiLemari === 1 ? '⚠ Tidak rapi · +1 poin' : 'Rapi · 0 poin'}
										</p>
									</div>
								</div>
								<div
									class="cb-box {nilaiSantri.rapiLemari === 1 ? 'is-checked' : ''}"
									aria-hidden="true"
								>
									{#if nilaiSantri.rapiLemari === 1}
										<svg
											xmlns="http://www.w3.org/2000/svg"
											class="h-3 w-3 text-white"
											fill="none"
											viewBox="0 0 24 24"
											stroke="currentColor"
											stroke-width="3.5"
										>
											<path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
										</svg>
									{/if}
								</div>
							</label>
						</div>
					{/if}
				</div>

				{#if selectedSantri}
					<div class="flex gap-2.5 px-5 pb-5">
						<button
							type="button"
							on:click={resetSantri}
							class="flex items-center gap-1.5 rounded-xl border border-gray-200 bg-white px-4 py-3 text-sm font-semibold text-gray-500 transition-all hover:bg-gray-50"
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
									d="M4 4v5h5M20 20v-5h-5M4.34 15a8 8 0 1015.32-6"
								/>
							</svg>
							Reset
						</button>
						<button
							type="button"
							on:click={submitSantri}
							disabled={isSubmitting}
							class="flex flex-1 items-center justify-center gap-2 rounded-xl bg-linear-to-r from-emerald-500 to-teal-600 py-3 text-sm font-bold text-white shadow-lg shadow-emerald-200 transition-all hover:from-emerald-600 hover:to-teal-700 hover:shadow-emerald-300 active:scale-[0.98] disabled:cursor-not-allowed disabled:opacity-50"
						>
							{#if isSubmitting}
								<svg
									class="h-4 w-4 animate-spin"
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									aria-hidden="true"
								>
									<circle
										class="opacity-25"
										cx="12"
										cy="12"
										r="10"
										stroke="currentColor"
										stroke-width="4"
									/>
									<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z" />
								</svg>
								Menyimpan…
							{:else}
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
										d="M8 7H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-3m-1 4l-3 3m0 0l-3-3m3 3V4"
									/>
								</svg>
								Simpan Penilaian
							{/if}
						</button>
					</div>
				{/if}
			</div>
		{/if}

		<!-- ══ KAMAR FORM ══ -->
		{#if activeTab === 'kamar'}
			<div class="fade-up overflow-visible rounded-3xl border border-gray-100 bg-white shadow-sm">
				<div class="border-b border-gray-50 px-5 pt-5 pb-4">
					<p class="font-bold text-gray-800">Penilaian Kamar</p>
					<p class="mt-0.5 text-xs text-gray-400">
						Cari kamar, lalu centang item yang <span class="font-semibold text-red-400"
							>tidak rapi</span
						>
					</p>
				</div>

				<div class="space-y-3 px-5 py-4">
					{#if !selectedKamar}
						<!-- Search kamar -->
						<!-- svelte-ignore a11y_no_static_element_interactions -->
						<div class="relative" on:click={stopAndClose} on:keydown={stopAndClose}>
							<span
								class="pointer-events-none absolute top-1/2 left-3.5 -translate-y-1/2 text-gray-400"
								aria-hidden="true"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									class="h-4 w-4"
									fill="none"
									viewBox="0 0 24 24"
									stroke="currentColor"
									stroke-width="2"
								>
									<circle cx="11" cy="11" r="8" /><path
										stroke-linecap="round"
										d="M21 21l-4.35-4.35"
									/>
								</svg>
							</span>
							<input
								class="w-full rounded-xl border border-gray-200 bg-gray-50 py-3 pr-4 pl-10 text-sm text-gray-800 placeholder-gray-300 transition-all focus:border-transparent focus:bg-white focus:ring-2 focus:ring-emerald-400 focus:outline-none"
								type="text"
								placeholder="Cari nomor kamar…"
								bind:value={kamarQuery}
								on:focus={() => (showKamarDropdown = true)}
								on:input={() => (showKamarDropdown = true)}
								autocomplete="off"
								aria-label="Cari kamar"
								aria-haspopup="listbox"
							/>
							{#if showKamarDropdown}
								<div class="dropdown" role="listbox" aria-label="Daftar kamar">
									{#if loadingKamar}
										<div class="px-4 py-3 text-center text-xs text-gray-400" role="status">
											Memuat data kamar…
										</div>
									{:else if filteredKamar.length === 0}
										<div class="px-4 py-3 text-center text-xs text-gray-400">
											Kamar tidak ditemukan
										</div>
									{:else}
										{#each filteredKamar as k}
											<button
												type="button"
												role="option"
												aria-selected="false"
												class="flex w-full items-center gap-3 px-4 py-2.5 text-left transition-colors hover:bg-emerald-50"
												on:click={() => selectKamar(k)}
											>
												<div
													class="flex h-8 w-8 shrink-0 items-center justify-center rounded-lg bg-linear-to-br from-teal-400 to-emerald-500"
													aria-hidden="true"
												>
													<svg
														xmlns="http://www.w3.org/2000/svg"
														class="h-4 w-4 text-white"
														fill="none"
														viewBox="0 0 24 24"
														stroke="currentColor"
														stroke-width="2"
													>
														<path
															stroke-linecap="round"
															stroke-linejoin="round"
															d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"
														/>
													</svg>
												</div>
												<p class="text-sm font-semibold text-gray-800">
													Kamar {k.nomor}
													{k.gedung}
												</p>
											</button>
										{/each}
									{/if}
								</div>
							{/if}
						</div>
					{:else}
						<!-- Selected kamar chip -->
						<div
							class="flex items-center gap-3 rounded-xl border border-emerald-100 bg-emerald-50 px-4 py-3"
						>
							<div
								class="flex h-8 w-8 shrink-0 items-center justify-center rounded-lg bg-linear-to-br from-teal-400 to-emerald-500"
								aria-hidden="true"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									class="h-4 w-4 text-white"
									fill="none"
									viewBox="0 0 24 24"
									stroke="currentColor"
									stroke-width="2"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"
									/>
								</svg>
							</div>
							<p class="flex-1 text-sm font-bold text-emerald-800">Kamar {selectedKamar.nomor}</p>
							<button
								type="button"
								on:click={resetKamar}
								aria-label="Ganti kamar"
								class="rounded-lg p-1 text-emerald-300 transition-colors hover:text-emerald-500"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									class="h-4 w-4"
									fill="none"
									viewBox="0 0 24 24"
									stroke="currentColor"
									stroke-width="2.5"
									aria-hidden="true"
								>
									<path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
								</svg>
							</button>
						</div>
					{/if}

					{#if selectedKamar}
						<div class="space-y-2 pt-1">
							<div class="flex items-center justify-between">
								<p class="text-xs font-semibold tracking-widest text-gray-500 uppercase">
									Item Penilaian
								</p>
								<span class="rounded-full bg-gray-100 px-2 py-0.5 text-xs text-gray-400">
									{nilaiKamar.lantai +
										nilaiKamar.kamarMandi +
										nilaiKamar.tempatSampah +
										nilaiKamar.gantunganBaju} poin
								</span>
							</div>

							{#each kamarItems as item}
								<label class="check-row {nilaiKamar[item.key] === 1 ? 'is-checked' : ''}">
									<input
										type="checkbox"
										class="sr-only"
										checked={nilaiKamar[item.key] === 1}
										on:change={(e) => toggleKamarNilai(item.key, e.currentTarget.checked)}
										aria-label="{item.label} tidak rapi"
									/>
									<div class="flex items-center gap-3">
										<span
											class="shrink-0 {nilaiKamar[item.key] === 1
												? 'text-red-400'
												: 'text-gray-400'}"
											aria-hidden="true"
										>
											<svg
												xmlns="http://www.w3.org/2000/svg"
												class="h-5 w-5"
												fill="none"
												viewBox="0 0 24 24"
												stroke="currentColor"
												stroke-width="1.8"
											>
												{#if item.key === 'lantai'}
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														d="M4 6h16M4 10h16M4 14h16M4 18h16"
													/>
												{:else if item.key === 'kamarMandi'}
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														d="M4 12h16M4 12V6a2 2 0 012-2h2a2 2 0 012 2v6M4 12v4a2 2 0 002 2h12a2 2 0 002-2v-4"
													/>
												{:else if item.key === 'tempatSampah'}
													<polyline points="3 6 5 6 21 6" />
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														d="M19 6l-1 14a2 2 0 01-2 2H8a2 2 0 01-2-2L5 6m5 0V4h4v2"
													/>
												{:else}
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														d="M20.59 13.41l-7.17 7.17a2 2 0 01-2.83 0L2 12V2h10l8.59 8.59a2 2 0 010 2.82z"
													/>
													<line x1="7" y1="7" x2="7.01" y2="7" />
												{/if}
											</svg>
										</span>
										<div>
											<p
												class="text-sm font-semibold {nilaiKamar[item.key] === 1
													? 'text-red-700'
													: 'text-gray-700'}"
											>
												{item.label}
											</p>
											<p
												class="text-xs {nilaiKamar[item.key] === 1
													? 'text-red-400'
													: 'text-gray-400'}"
											>
												{nilaiKamar[item.key] === 1
													? '⚠ Tidak rapi · +1 poin'
													: `${item.sub} · 0 poin`}
											</p>
										</div>
									</div>
									<div
										class="cb-box {nilaiKamar[item.key] === 1 ? 'is-checked' : ''}"
										aria-hidden="true"
									>
										{#if nilaiKamar[item.key] === 1}
											<svg
												xmlns="http://www.w3.org/2000/svg"
												class="h-3 w-3 text-white"
												fill="none"
												viewBox="0 0 24 24"
												stroke="currentColor"
												stroke-width="3.5"
											>
												<path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
											</svg>
										{/if}
									</div>
								</label>
							{/each}
						</div>
					{/if}
				</div>

				{#if selectedKamar}
					<div class="flex gap-2.5 px-5 pb-5">
						<button
							type="button"
							on:click={resetKamar}
							class="flex items-center gap-1.5 rounded-xl border border-gray-200 bg-white px-4 py-3 text-sm font-semibold text-gray-500 transition-all hover:bg-gray-50"
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
									d="M4 4v5h5M20 20v-5h-5M4.34 15a8 8 0 1015.32-6"
								/>
							</svg>
							Reset
						</button>
						<button
							type="button"
							on:click={submitKamar}
							disabled={isSubmitting}
							class="flex flex-1 items-center justify-center gap-2 rounded-xl bg-linear-to-r from-emerald-500 to-teal-600 py-3 text-sm font-bold text-white shadow-lg shadow-emerald-200 transition-all hover:from-emerald-600 hover:to-teal-700 hover:shadow-emerald-300 active:scale-[0.98] disabled:cursor-not-allowed disabled:opacity-50"
						>
							{#if isSubmitting}
								<svg
									class="h-4 w-4 animate-spin"
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									aria-hidden="true"
								>
									<circle
										class="opacity-25"
										cx="12"
										cy="12"
										r="10"
										stroke="currentColor"
										stroke-width="4"
									/>
									<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z" />
								</svg>
								Menyimpan…
							{:else}
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
										d="M8 7H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-3m-1 4l-3 3m0 0l-3-3m3 3V4"
									/>
								</svg>
								Simpan Penilaian
							{/if}
						</button>
					</div>
				{/if}
			</div>
		{/if}

		<!-- Legend -->
		<div class="flex items-center justify-center gap-5 pt-1 text-xs text-gray-400">
			<span class="flex items-center gap-1.5">
				<span
					class="flex h-4 w-4 shrink-0 items-center justify-center rounded border-2 border-red-400 bg-red-400"
					aria-hidden="true"
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-2.5 w-2.5 text-white"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
						stroke-width="3.5"
					>
						<path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
					</svg>
				</span>
				Centang = tidak rapi (+1 poin)
			</span>
			<span class="flex items-center gap-1.5">
				<span
					class="h-4 w-4 shrink-0 rounded border-2 border-gray-300 bg-white"
					aria-hidden="true"
				></span>
				Kosong = rapi (0 poin)
			</span>
		</div>
	</main>
</div>

<style>
	:global(body) {
		font-family: 'DM Sans', sans-serif;
	}

	.check-row {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0.85rem 1rem;
		border-radius: 0.875rem;
		border: 1.5px solid #e5e7eb;
		background: #fafafa;
		cursor: pointer;
		user-select: none;
		transition: all 0.15s;
		width: 100%;
	}
	.check-row:hover {
		border-color: #d1d5db;
		background: #f3f4f6;
	}
	.check-row.is-checked {
		background: #fff1f2;
		border-color: #fecdd3;
	}

	.cb-box {
		width: 1.2rem;
		height: 1.2rem;
		border-radius: 0.35rem;
		border: 2px solid #d1d5db;
		background: white;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.15s;
		flex-shrink: 0;
	}
	.cb-box.is-checked {
		background: #ef4444;
		border-color: #ef4444;
	}

	.dropdown {
		position: absolute;
		top: calc(100% + 5px);
		left: 0;
		right: 0;
		z-index: 50;
		background: white;
		border: 1.5px solid #e5e7eb;
		border-radius: 0.875rem;
		box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
		overflow: hidden;
		animation: dropIn 0.15s ease;
	}
	@keyframes dropIn {
		from {
			opacity: 0;
			transform: translateY(-5px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	@keyframes fadeUp {
		from {
			opacity: 0;
			transform: translateY(10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
	.fade-up {
		animation: fadeUp 0.25s ease;
	}

	@keyframes alertIn {
		from {
			opacity: 0;
			transform: translateY(-6px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
	.alert-in {
		animation: alertIn 0.2s ease;
	}
</style>
