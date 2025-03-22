<script lang="ts">
	import { onMount } from 'svelte';
	import Login from '../components/Login.svelte';
	import type { Playlist } from '../types';

	let playlists: Playlist[] = [];
	let accessToken: string | null = null;
	let checkedPlaylists: boolean[] = []; // Use an array for checked state
	let loading: boolean = true;
	let isDeleting: boolean = false; // Add this to track deletion progress
	let userProfile: { display_name: string; images: { url: string }[] } | null = null;

	onMount(() => {
		accessToken = localStorage.getItem('spotify_access_token');

		if (accessToken) {
			fetchPlaylists(accessToken);
			fetchUserProfile(accessToken);
		}
	});

	async function fetchPlaylists(token: string) {
		try {
			let allPlaylists: Playlist[] = [];
			let url = 'https://api.spotify.com/v1/me/playlists?limit=50'; // Maximum limit allowed

			while (url) {
				const response = await fetch(url, {
					headers: {
						Authorization: `Bearer ${token}`
					}
				});

				if (response.ok) {
					const data = await response.json();
					console.log('Page of playlist data:', data);
					allPlaylists = [...allPlaylists, ...data.items];

					// Get the URL for the next page or set to null if no more pages
					url = data.next;
				} else {
					console.error('Failed to fetch playlists:', response.statusText);
					break;
				}
			}

			playlists = allPlaylists;
			console.log('Total playlists fetched:', playlists.length);
			loading = false;
			checkedPlaylists = new Array(playlists.length).fill(false); // Initialize array
		} catch (error) {
			console.error('Error fetching playlists:', error);
		}
	}

	async function fetchUserProfile(token: string) {
		try {
			const response = await fetch('https://api.spotify.com/v1/me', {
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (response.ok) {
				userProfile = await response.json();
				console.log('User profile:', userProfile);
			} else {
				console.error('Failed to fetch user profile:', response.statusText);
			}
		} catch (error) {
			console.error('Error fetching user profile:', error);
		}
	}

	function logout() {
		localStorage.removeItem('spotify_access_token');
		window.location.reload();
	}

	function logCheckedItems() {
		const checkedItems = playlists.filter((_, index) => checkedPlaylists[index]);
		console.log('Checked Playlists:', checkedItems);
		return checkedItems;
	}

	function selectAll() {
		checkedPlaylists = new Array(playlists.length).fill(true);
		logCheckedItems();
	}

	function clearAll() {
		checkedPlaylists = new Array(playlists.length).fill(false);
		logCheckedItems();
	}

	function logChange() {
		console.log('Checked Playlists Array:', checkedPlaylists);
		logCheckedItems();
	}

	// Add this function for deleting playlists
	async function deleteSelectedPlaylists() {
		if (!accessToken) return;

		const selectedPlaylists = logCheckedItems();

		if (selectedPlaylists.length === 0) {
			alert('No playlists selected');
			return;
		}

		if (!confirm(`Are you sure you want to delete ${selectedPlaylists.length} playlists?`)) {
			return;
		}

		isDeleting = true;
		let successCount = 0;
		let errorCount = 0;

		for (const playlist of selectedPlaylists) {
			try {
				// Spotify uses DELETE method for unfollowing (removing) playlists
				const response = await fetch(
					`https://api.spotify.com/v1/playlists/${playlist.id}/followers`,
					{
						method: 'DELETE',
						headers: {
							Authorization: `Bearer ${accessToken}`
						}
					}
				);

				if (response.ok) {
					successCount++;
					console.log(`Successfully deleted playlist: ${playlist.name}`);
				} else {
					console.error(`Failed to delete playlist ${playlist.name}: ${response.statusText}`);
					errorCount++;
				}
			} catch (error) {
				console.error(`Error deleting playlist ${playlist.name}:`, error);
				errorCount++;
			}
		}

		alert(`Deleted ${successCount} playlists successfully. ${errorCount} failed.`);
		isDeleting = false;

		// Refresh the playlist list after deletion
		await fetchPlaylists(accessToken);
	}
</script>

{#if !accessToken}
	<div class="flex h-screen items-center justify-center bg-black">
		<Login />
	</div>
{:else if loading}
	<div class="flex h-screen items-center justify-center bg-black">
		<div class="flex flex-col items-center justify-center gap-5">
			<div
				class="mt-4 h-8 w-8 animate-spin rounded-full border-4 border-white border-t-transparent"
			></div>
			<h1 class="text-2xl font-bold text-white">Loading Your Playlists...</h1>
		</div>
	</div>
{:else if isDeleting}
	<div class="flex h-screen items-center justify-center bg-black">
		<div class="flex flex-col items-center justify-center gap-5">
			<div
				class="mt-4 h-8 w-8 animate-spin rounded-full border-4 border-white border-t-transparent"
			></div>
			<h1 class="text-2xl font-bold text-white">Deleting Selected Playlists...</h1>
		</div>
	</div>
{:else}
	<div class="flex h-screen flex-col bg-black text-white">
		<div class="flex w-full items-center justify-between bg-black p-4">
			{#if userProfile}
				<div class="flex items-center space-x-4">
					{#if userProfile.images && userProfile.images.length > 0}
						<!-- svelte-ignore a11y_img_redundant_alt -->
						<img
							src={userProfile.images[0].url}
							alt="Profile Picture"
							class="h-12 w-12 rounded-full"
						/>
					{/if}
					<h2 class="text-xl font-bold">{userProfile.display_name}</h2>
				</div>
			{/if}
			<div>
				<h1 class="text-2xl font-bold">Select playlists to delete</h1>
				<h6 class="text-center text-sm">
					{playlists.length} playlists found
				</h6>
			</div>
			<div class="flex gap-8 space-x-2">
				<div class="flex space-x-2">
					<button
						on:click={selectAll}
						class="rounded bg-green-500 px-2 py-1 text-white hover:cursor-pointer hover:bg-green-400"
					>
						Select All
					</button>
					<button
						on:click={clearAll}
						class="rounded bg-red-500 px-2 py-1 text-white hover:cursor-pointer hover:bg-red-400"
					>
						Clear All
					</button>
					<!-- Add Delete Selected button -->
					<button
						on:click={deleteSelectedPlaylists}
						class="rounded bg-red-600 px-2 py-1 text-white hover:cursor-pointer hover:bg-red-700"
					>
						Delete Selected
					</button>
				</div>
				<button
					on:click={logout}
					class="rounded bg-yellow-500 px-2 py-1 text-white hover:cursor-pointer hover:bg-yellow-400"
				>
					Log Out
				</button>
			</div>
		</div>
		<div class="flex-grow overflow-auto p-4">
			<div class="grid grid-cols-1 gap-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
				{#each playlists as playlist, index}
					<div
						class="cursor-pointer rounded bg-white from-purple-500 via-pink-500 to-red-500 p-4 text-black shadow-md hover:bg-gradient-to-r"
					>
						<input
							type="checkbox"
							class="mr-2 rounded-sm"
							bind:checked={checkedPlaylists[index]}
							on:change={logChange}
						/>
						<h2 class="inline-block text-lg font-semibold">
							{playlist.name.trim() === '' ? 'No Name' : playlist.name}
						</h2>
						<h6>{playlist.tracks.total} tracks</h6>
					</div>
				{/each}
			</div>
		</div>
	</div>
{/if}
