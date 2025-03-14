<script>
	import { onMount } from 'svelte';

	onMount(async () => {
		const urlParams = new URLSearchParams(window.location.search);
		const code = urlParams.get('code');

		if (code) {
			try {
				// Exchange the authorization code for an access token
				const response = await fetch(`http://localhost:3000/callback?code=${code}`);
				const data = await response.json();
				const accessToken = data.access_token;

				// Store the access token (e.g., in local storage)
				localStorage.setItem('spotify_access_token', accessToken);

				// Redirect to the main page
				window.location.href = '/';
			} catch (error) {
				console.error('Error exchanging code for token:', error);
			}
		} else {
			console.error('Authorization code not found in URL');
		}
	});
</script>

<div
	class="flex h-screen items-center justify-center bg-gradient-to-r from-purple-500 via-pink-500 to-red-500"
>
	<div class="text-center text-white">
		<h1 class="animate-pulse text-4xl font-extrabold">Authenticating...</h1>
		<p class="mt-4 text-lg">Please wait while we complete the authentication process.</p>
	</div>
</div>
