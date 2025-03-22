<script lang="ts">
	import { onMount } from 'svelte';
	import Login from '../components/Login.svelte';
	import type { Playlist } from '../types';

	let playlists: Playlist[] = [];
	let accessToken: string | null = null;
	let checkedPlaylists: boolean[] = []; // Use an array for checked state
	let loading: boolean = true;

	onMount(() => {
		accessToken = localStorage.getItem('spotify_access_token');

		if (accessToken) {
			fetchPlaylists(accessToken);
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

	function logout() {
		localStorage.removeItem('spotify_access_token');
		window.location.reload();
	}

	function logCheckedItems() {
		const checkedItems = playlists.filter((_, index) => checkedPlaylists[index]);
		console.log('Checked Playlists:', checkedItems);
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
</script>

{#if !accessToken}
	<div class="flex h-screen items-center justify-center">
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
{:else}
	<div class="flex flex-col items-center justify-center bg-black text-white">
		<div class="flex w-full items-center justify-between p-4">
			<h1 class="text-2xl font-bold">Select playlists to delete</h1>
			<h6 class="text-sm">
				{playlists.length} playlists found
			</h6>
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
				<button
					on:click={logout}
					class="rounded bg-yellow-500 px-2 py-1 text-white hover:cursor-pointer hover:bg-yellow-400"
				>
					Log Out
				</button>
			</div>
		</div>
		<div class="grid grid-cols-1 gap-4 p-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
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
{/if}
