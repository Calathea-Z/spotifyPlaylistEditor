<script lang="ts">
	import { onMount } from 'svelte';
	import Login from '../components/Login.svelte';
	import type { Playlist } from '../types';

	let playlists: Playlist[] = [];
	let accessToken: string | null = null;

	onMount(() => {
		accessToken = localStorage.getItem('spotify_access_token');

		if (accessToken) {
			fetchPlaylists(accessToken);
		}
	});

	async function fetchPlaylists(token: string) {
		try {
			const response = await fetch('https://api.spotify.com/v1/me/playlists', {
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (response.ok) {
				const data = await response.json();
				playlists = data.items;
			} else {
				console.error('Failed to fetch playlists:', response.statusText);
			}
		} catch (error) {
			console.error('Error fetching playlists:', error);
		}
	}

	function logout() {
		localStorage.removeItem('spotify_access_token');
		window.location.reload();
	}
</script>

{#if !accessToken}
	<div class="flex h-screen items-center justify-center">
		<Login />
	</div>
{:else}
	<div class="flex flex-col items-center justify-center bg-black text-white">
		<div class="flex w-full items-center justify-between p-4">
			<h1 class="text-2xl font-bold">Select playlists to delete</h1>
			<button
				on:click={logout}
				class="rounded bg-yellow-500 px-2 py-1 text-white hover:cursor-pointer hover:bg-yellow-400"
			>
				Log Out
			</button>
		</div>
		<div class="grid grid-cols-1 gap-4 p-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
			{#each playlists as playlist}
				<div
					class="cursor-pointer rounded bg-white from-purple-500 via-pink-500 to-red-500 p-4 text-black shadow-md hover:bg-gradient-to-r"
				>
					<h2 class="text-lg font-semibold">
						{playlist.name === ' ' ? 'No Name' : playlist.name}
					</h2>
					<h6>{playlist.tracks.total} tracks</h6>
				</div>
			{/each}
		</div>
	</div>
{/if}
