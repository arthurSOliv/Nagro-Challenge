<template>
	<div id="app">
		<Header title="Bem vindo" :hideUserDropdown="!user" />
		<Menu />
		<Loading v-if="validatingToken" />
		<Content v-else />
		<Footer />
	</div>
</template>

<script>
import axios from 'axios';
import { baseApiUrl, userKey } from '@/global';
import Header from "@/components/template/Header";
import Menu from "@/components/template/Menu";
import Footer from "@/components/template/Footer";
import Content from "@/components/template/Content";
import Loading from "@/components/template/Loading";
import { mapState } from 'vuex';

export default {
	name: "App",
	components: { Header, Menu, Footer, Content, Loading },
	computed: mapState(['user']),
	data: function() {
		return {
			validatingToken: true
		}
	},
	methods: {
		async validateToken() {
			this.validatingToken = true;

			const json = localStorage.getItem(userKey);
			const userData = JSON.parse(json);
			this.$store.commit('setUser', null);

			if(!userData) {
				this.validatingToken = false;
				this.$router.push({ name: 'auth' });
				return;
			}

			const res = await axios.post(`${baseApiUrl}/validateToken`, userData);

			if(res.data) {
				this.$store.commit('setUser', userData);
			} else{
				localStorage.removeItem(userKey);
				this.$router.push({ name: 'auth' });
			}

			this.validatingToken = false;
		}
	},
	created() {
		this.validateToken();
	}
}
</script>

<style>
	* {
		font-family: "Lato", sans-serif;
	}

	body {
		margin: 0;
	}

	#app {
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;

		height: 100vh;
		display: grid;
		grid-template-rows: 60px 1fr 40px;
		grid-template-columns: 300px 1fr;
		grid-template-areas:
			"header header"
			"content content"
			"footer footer"
		;
	}

	#app.hide-menu {
		grid-template-areas:
			"header header"
			"content content"
			"footer footer";
	}
</style>