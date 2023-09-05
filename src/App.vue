<template>
	<router-view />
</template>


<script lang="ts">
import { defineComponent } from 'vue';
import axios from 'axios';
import state from './state.js';

export default defineComponent({
	name: 'App',
	async created() {
		axios.defaults.baseURL = 'https://corsproxy.io/?' + encodeURIComponent('https://fall.xtds.dance/api/v1');
		axios.defaults.headers = {
			'Content-Type': 'application/json',
		};
		if (window.localStorage.getItem('refresh_token')) {
			const response = await axios.post('/auth/refresh', {
				refresh_token: window.localStorage.getItem('refresh_token'),
			});
			window.localStorage.setItem('access_token', response.data.access_token);
			window.localStorage.setItem('refresh_token', response.data.refresh_token);
			axios.defaults.headers.Authorization = 'Bearer ' + response.data.access_token;
			state.loggedIn = true;
		}
	},
});
</script>

<style src="./styles/app.scss" lang="scss"/>


