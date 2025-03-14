  <script>
    function login() {
      window.location.href = 'http://localhost:3000/login';
    }
  </script>

  <button on:click={login} class="bg-blue-500 text-white px-4 py-2 rounded hover:cursor-pointer hover:bg-red-200">Login with Spotify</button>