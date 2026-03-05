<script lang="ts">
	import '../layout.css';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';

	let menuOpen = false;
	let showLogoutConfirm = false;

	const navLinks = [
		{
			href: '/main-page/dashboard',
			label: 'Dashboard',
			icon: `<path stroke-linecap="round" stroke-linejoin="round" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"/>`
		},
		{
			href: '/main-page/input-penilaian',
			label: 'Input Penilaian',
			icon: `<path stroke-linecap="round" stroke-linejoin="round" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>`
		}
	];

	function closeMenu(): void {
		menuOpen = false;
	}

	function confirmLogout(): void {
		showLogoutConfirm = true;
		menuOpen = false;
	}

	function cancelLogout(): void {
		showLogoutConfirm = false;
	}

	function doLogout(): void {
		localStorage.removeItem('token');
		localStorage.removeItem('nama');
		showLogoutConfirm = false;
		goto('/');
	}
</script>

<svelte:window on:click={closeMenu} />

<!-- ── Logout Confirm Modal ───────────────────────────────────────────── -->
{#if showLogoutConfirm}
	<!-- svelte-ignore a11y_no_static_element_interactions -->
	<div
		class="fixed inset-0 z-[100] flex items-center justify-center p-4"
		on:click={cancelLogout}
		on:keydown={(e) => e.key === 'Escape' && cancelLogout()}
		role="dialog"
		aria-modal="true"
		aria-labelledby="logout-title"
	>
		<!-- Backdrop -->
		<div class="absolute inset-0 bg-black/30 backdrop-blur-sm"></div>

		<!-- Dialog card -->
		<!-- svelte-ignore a11y_no_static_element_interactions -->
		<div
			class="relative w-full max-w-sm rounded-3xl border border-gray-100 bg-white p-6 shadow-2xl shadow-black/10"
			on:click|stopPropagation
			on:keydown|stopPropagation
		>
			<!-- Icon -->
			<div class="mx-auto mb-4 flex h-14 w-14 items-center justify-center rounded-2xl bg-red-50">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-7 w-7 text-red-500"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
					stroke-width="1.8"
					aria-hidden="true"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"
					/>
				</svg>
			</div>

			<!-- Text -->
			<h2 id="logout-title" class="text-center text-base font-bold text-gray-800">
				Keluar dari Akun?
			</h2>
			<p class="mt-1.5 text-center text-sm text-gray-400">
				Sesi kamu akan diakhiri dan token akan dihapus.
			</p>

			<!-- Actions -->
			<div class="mt-5 flex gap-2.5">
				<button
					type="button"
					on:click={cancelLogout}
					class="flex-1 rounded-xl border border-gray-200 bg-white py-3 text-sm font-semibold text-gray-600 transition-all hover:bg-gray-50"
				>
					Batal
				</button>
				<button
					type="button"
					on:click={doLogout}
					class="flex-1 rounded-xl bg-red-500 py-3 text-sm font-semibold text-white shadow-md shadow-red-200 transition-all hover:bg-red-600 active:scale-[0.98]"
				>
					Ya, Keluar
				</button>
			</div>
		</div>
	</div>
{/if}

<!-- ── Layout ────────────────────────────────────────────────────────── -->
<div class="flex min-h-screen flex-col">
	<header class="sticky top-0 z-50 bg-emerald-600 text-white shadow-lg shadow-emerald-900/20">
		<div class="mx-auto flex h-14 max-w-5xl items-center justify-between px-4">
			<!-- Logo / Title -->
			<div class="flex items-center gap-2.5">
				<div class="flex h-7 w-7 shrink-0 items-center justify-center rounded-lg bg-white/20">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-4 w-4 text-white"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
						stroke-width="2"
						aria-hidden="true"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							d="M3 10.5V21h6v-5.25a2.25 2.25 0 014.5 0V21H21V10.5M12 3L2 10h20L12 3z"
						/>
					</svg>
				</div>
				<span class="text-sm font-bold tracking-wide">Sistem Penilaian 3B</span>
			</div>

			<!-- Desktop nav + logout -->
			<div class="hidden items-center gap-1 sm:flex">
				<nav class="flex items-center gap-1" aria-label="Navigasi utama">
					{#each navLinks as link}
						{@const isActive = $page.url.pathname === link.href}
						<a
							href={link.href}
							class="flex items-center gap-1.5 rounded-xl px-3.5 py-2 text-sm font-medium transition-all duration-150
								{isActive
								? 'bg-white font-semibold text-emerald-700 shadow-sm'
								: 'text-white/80 hover:bg-white/15 hover:text-white'}"
							aria-current={isActive ? 'page' : undefined}
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
								{@html link.icon}
							</svg>
							{link.label}
						</a>
					{/each}
				</nav>

				<!-- Divider -->
				<div class="mx-1.5 h-5 w-px bg-white/20"></div>

				<!-- Logout button desktop -->
				<button
					type="button"
					on:click|stopPropagation={confirmLogout}
					aria-label="Logout"
					class="flex items-center gap-1.5 rounded-xl px-3 py-2 text-sm font-medium text-white/70 transition-all hover:bg-red-500/20 hover:text-white"
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
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"
						/>
					</svg>
					Keluar
				</button>
			</div>

			<!-- Hamburger (mobile) -->
			<button
				type="button"
				class="flex h-9 w-9 items-center justify-center rounded-xl bg-white/15 transition-colors hover:bg-white/25 sm:hidden"
				on:click|stopPropagation={() => (menuOpen = !menuOpen)}
				aria-label={menuOpen ? 'Tutup menu' : 'Buka menu'}
				aria-expanded={menuOpen}
				aria-controls="mobile-menu"
			>
				{#if menuOpen}
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-5 w-5"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
						stroke-width="2.5"
						aria-hidden="true"
					>
						<path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
					</svg>
				{:else}
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-5 w-5"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
						stroke-width="2.5"
						aria-hidden="true"
					>
						<path stroke-linecap="round" stroke-linejoin="round" d="M4 6h16M4 12h16M4 18h16" />
					</svg>
				{/if}
			</button>
		</div>

		<!-- Mobile dropdown -->
		{#if menuOpen}
			<div
				id="mobile-menu"
				class="space-y-1 border-t border-white/10 bg-emerald-700 px-3 py-2 sm:hidden"
				role="menu"
				aria-label="Menu navigasi mobile"
			>
				{#each navLinks as link}
					{@const isActive = $page.url.pathname === link.href}
					<a
						href={link.href}
						role="menuitem"
						class="flex items-center gap-3 rounded-xl px-3 py-2.5 text-sm font-medium transition-all
							{isActive
							? 'bg-white font-semibold text-emerald-700'
							: 'text-white/80 hover:bg-white/15 hover:text-white'}"
						aria-current={isActive ? 'page' : undefined}
						on:click={closeMenu}
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
							{@html link.icon}
						</svg>
						{link.label}
						{#if isActive}
							<span class="ml-auto h-1.5 w-1.5 shrink-0 rounded-full bg-emerald-400"></span>
						{/if}
					</a>
				{/each}

				<!-- Logout mobile -->
				<div class="mt-1 border-t border-white/10 pt-1">
					<button
						type="button"
						role="menuitem"
						on:click={confirmLogout}
						class="flex w-full items-center gap-3 rounded-xl px-3 py-2.5 text-sm font-medium text-red-300 transition-all hover:bg-red-500/20 hover:text-red-200"
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
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"
							/>
						</svg>
						Keluar
					</button>
				</div>
			</div>
		{/if}
	</header>

	<main
		class="mx-auto mt-6 w-full max-w-5xl rounded-lg bg-white/80 p-6 shadow-lg shadow-emerald-200"
	>
		<slot />
	</main>
</div>
