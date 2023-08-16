<script setup>
    const AUTH_URL = "https://accounts.spotify.com/authorize?" + 
    new URLSearchParams({
        response_type: 'code',
        client_id: 'dde63debce154f3f85fa86c5b6e43ddb',
        scope: 'user-read-email user-library-read user-read-playback-state user-modify-playback-state',
        redirect_uri: 'http://localhost:5173'
    }).toString();
</script>

<template>
    <!-- <a :href="process.env.AUTH_URL"></a> -->
    <a :href="AUTH_URL">Login</a>
</template>
