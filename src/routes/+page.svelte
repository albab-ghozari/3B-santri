<script lang="ts">
	import './layout.css';
	let username = '';
	let password = '';
	let showPassword = false;
	let isLoading = false;
	let errorMessage = '';
	const baseUrl = import.meta.env.API_BASE_URL

	async function handleLogin() {
		if (!username || !password) {
			errorMessage = 'Username dan password wajib diisi.';
			return;
		}

		isLoading = true;
		errorMessage = '';

		try {
			const res = await fetch(`${baseUrl}/api/login`, {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ username, password })
			});

			const data = await res.json();

			if (!res.ok) {
				errorMessage = data.message || 'Terjadi kesalahan.';
			} else {
				localStorage.setItem('token', data.token);
				localStorage.setItem('nama', data.nama);
				window.location.href = '/main-page/input-penilaian'; // Redirect ke dashboard setelah login sukses
			}
		} catch (e) {
			errorMessage = 'Tidak dapat terhubung ke server.';
		} finally {
			isLoading = false;
		}
	}

	function handleKeydown(e: KeyboardEvent) {
		if (e.key === 'Enter') handleLogin();
	}
</script>

<div
	class="min-h-screen flex flex-col items-center justify-center bg-linear-to-br from-emerald-50 via-white to-teal-50 p-4 relative overflow-hidden"
>
	<!-- Decorative blobs -->
	<div
		class="absolute top-0 left-0 h-72 w-72 -translate-x-1/2 -translate-y-1/2 rounded-full bg-emerald-100 opacity-50 mix-blend-multiply blur-3xl filter"
	></div>
	<div
		class="absolute right-0 bottom-0 h-96 w-96 translate-x-1/3 translate-y-1/3 rounded-full bg-teal-100 opacity-50 mix-blend-multiply blur-3xl filter"
	></div>

	<div class="relative w-full max-w-md">
		<!-- Card -->
		<div
			class="rounded-3xl border border-white/60 bg-white/80 p-8 shadow-xl shadow-emerald-100/60 backdrop-blur-xl sm:p-10"
		>
			<!-- Header -->
			<div class="mb-8 flex flex-col items-center">
				<!-- Logo / Icon -->
				<div
					class="mb-5 flex h-16 w-16 items-center justify-center rounded-2xl bg-linear-to-br from-emerald-500 to-teal-600 shadow-lg shadow-emerald-200"
				>
					<!-- Mosque / Islamic star icon (inline SVG) -->
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-8 w-8 text-white"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
						stroke-width="1.8"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							d="M3 10.5V21h6v-5.25a2.25 2.25 0 014.5 0V21H21V10.5M12 3L2 10h20L12 3z"
						/>
					</svg>
				</div>
				<h1 class="text-2xl font-bold tracking-tight text-gray-800">Portal Admin</h1>
				<p class="mt-1 text-sm text-gray-400">Sistem Manajemen Pesantren</p>
			</div>

			<!-- Error Alert -->
			{#if errorMessage}
				<div
					class="mb-6 flex items-start gap-3 rounded-2xl border border-red-100 bg-red-50 px-4 py-3 text-sm text-red-600"
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="mt-0.5 h-4 w-4 shrink-0"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
						stroke-width="2"
					>
						<circle cx="12" cy="12" r="10" /><line x1="12" y1="8" x2="12" y2="12" /><line
							x1="12"
							y1="16"
							x2="12.01"
							y2="16"
						/>
					</svg>
					<span>{errorMessage}</span>
				</div>
			{/if}

			<!-- Form -->
			<div class="space-y-5">
				<!-- Username -->
				<div class="group">
					<label
						class="mb-2 block text-xs font-semibold tracking-widest text-gray-500 uppercase"
						for="username"
					>
						Username
					</label>
					<div class="relative">
						<span
							class="absolute top-1/2 left-4 -translate-y-1/2 text-gray-400 transition-colors group-focus-within:text-emerald-500"
						>
							<svg
								xmlns="http://www.w3.org/2000/svg"
								class="h-4 w-4"
								fill="none"
								viewBox="0 0 24 24"
								stroke="currentColor"
								stroke-width="2"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"
								/>
							</svg>
						</span>
						<input
							id="username"
							type="text"
							bind:value={username}
							on:keydown={handleKeydown}
							placeholder="Masukkan username"
							autocomplete="username"
							class="w-full rounded-xl border border-gray-200 bg-gray-50 py-3 pr-4 pl-11 text-sm text-gray-800 placeholder-gray-300
                     transition-all duration-200 focus:border-transparent focus:bg-white focus:ring-2
                     focus:ring-emerald-400 focus:outline-none"
						/>
					</div>
				</div>

				<!-- Password -->
				<div class="group">
					<label
						class="mb-2 block text-xs font-semibold tracking-widest text-gray-500 uppercase"
						for="password"
					>
						Password
					</label>
					<div class="relative">
						<span
							class="absolute top-1/2 left-4 -translate-y-1/2 text-gray-400 transition-colors group-focus-within:text-emerald-500"
						>
							<svg
								xmlns="http://www.w3.org/2000/svg"
								class="h-4 w-4"
								fill="none"
								viewBox="0 0 24 24"
								stroke="currentColor"
								stroke-width="2"
							>
								<rect x="3" y="11" width="18" height="11" rx="2" ry="2" />
								<path stroke-linecap="round" stroke-linejoin="round" d="M7 11V7a5 5 0 0110 0v4" />
							</svg>
						</span>
						<input
							id="password"
							type={showPassword ? 'text' : 'password'}
							bind:value={password}
							on:keydown={handleKeydown}
							placeholder="Masukkan password"
							autocomplete="current-password"
							class="w-full rounded-xl border border-gray-200 bg-gray-50 py-3 pr-12 pl-11 text-sm text-gray-800 placeholder-gray-300
                     transition-all duration-200 focus:border-transparent focus:bg-white focus:ring-2
                     focus:ring-emerald-400 focus:outline-none"
						/>
						<button
							type="button"
							on:click={() => (showPassword = !showPassword)}
							class="absolute top-1/2 right-4 -translate-y-1/2 text-gray-400 transition-colors hover:text-emerald-500"
							tabindex="-1"
							aria-label="Toggle password visibility"
						>
							{#if showPassword}
								<!-- Eye off -->
								<svg
									xmlns="http://www.w3.org/2000/svg"
									class="h-4 w-4"
									fill="none"
									viewBox="0 0 24 24"
									stroke="currentColor"
									stroke-width="2"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										d="M17.94 17.94A10.07 10.07 0 0112 20c-7 0-11-8-11-8a18.45 18.45 0 015.06-5.94M9.9 4.24A9.12 9.12 0 0112 4c7 0 11 8 11 8a18.5 18.5 0 01-2.16 3.19m-6.72-1.07a3 3 0 11-4.24-4.24"
									/>
									<line x1="1" y1="1" x2="23" y2="23" />
								</svg>
							{:else}
								<!-- Eye -->
								<svg
									xmlns="http://www.w3.org/2000/svg"
									class="h-4 w-4"
									fill="none"
									viewBox="0 0 24 24"
									stroke="currentColor"
									stroke-width="2"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										d="M1 12S5 4 12 4s11 8 11 8-4 8-11 8S1 12 1 12z"
									/>
									<circle cx="12" cy="12" r="3" />
								</svg>
							{/if}
						</button>
					</div>
				</div>

				<!-- Submit Button -->
				<button
					on:click={handleLogin}
					disabled={isLoading}
					class="mt-2 flex w-full items-center justify-center gap-2 rounded-xl bg-linear-to-r from-emerald-500
				 to-teal-600 py-3.5 text-sm font-semibold text-white
				 shadow-lg shadow-emerald-200 transition-all duration-200 hover:from-emerald-600 hover:to-teal-700 hover:shadow-emerald-300
				 active:scale-[0.98] disabled:cursor-not-allowed disabled:opacity-60 disabled:active:scale-100"
				>
					{#if isLoading}
						<!-- Spinner -->
						<svg
							class="h-4 w-4 animate-spin text-white"
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
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
						<span>Memproses...</span>
					{:else}
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-4 w-4"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
							stroke-width="2"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								d="M11 16l-4-4m0 0l4-4m-4 4h14m-5 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h7a3 3 0 013 3v1"
							/>
						</svg>
						<span>Masuk</span>
					{/if}
				</button>
			</div>

			<!-- Footer -->
			<p class="mt-8 text-center text-xs text-gray-300">
				© {new Date().getFullYear()} Sistem Manajemen Pesantren
			</p>
		</div>
	</div>
</div>
