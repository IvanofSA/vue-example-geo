<template>
	<div class="signup container">
		<form class="card-panel" @submit.prevent="signup">
			<h2 class="center deep-purple-text">Signup</h2>
			<div class="field">
				<label for="email">Email</label>
				<input id="email" type="email" v-model="email">
			</div>
			<div class="field">
				<label for="password">Password</label>
				<input id="password" type="password" v-model="password">
			</div>
			<div class="field">
				<label for="name">Alias</label>
				<input id="name" type="text" v-model="alias">
			</div>
			<p class="red-text center" v-if="feedback">{{ feedback }}</p>
			<div class="field center">
				<button class="btn deep-purple">Signup</button>
			</div>
		</form>
	</div>
</template>

<script>
	import slugify from 'slugify'
	import db from '@/firebase/init'
	import firebase from 'firebase'

	export default {
		name: 'Signup',
		data() {
			return {
				email: null,
				password: null,
				alias: null,
				feedback: null,
				slug: null
			}
		},
		methods: {
			signup() {
				if(this.alias && this.email && this.password) {
					this.slug = slugify(this.alias, {
						replacement: '-',
						remove: /[$*_+~.()'"!\-:@]/g,
						lower: true
					})

					let ref = db.collection('users').doc(this.slug)

					ref.get().then(doc => {
						if(doc.exists) {
							this.feedback = 'This alias already exists'
						} else {
							firebase.auth().createUserWithEmailAndPassword(this.email, this.password)
							// firebase.auth().signInWithEmailAndPassword(this.email, this.password)
								.then(cred => {

									ref.set({
										alias: this.alias,
										geolocation: null,
										user_id: cred.user.uid
									})
									console.log(cred);
								})
								.then(() => {
									this.$router.push({name: 'GMap'})
								})
								.catch(err => {
									console.log(err);
									this.feedback = err.message
								})
							this.feedback = 'This alias is free to use'
						}
					})

				} else {
					this.feedback = 'You must enter all fields'
				}
			}
		}
	}
</script>

<style>
	.signup {
		max-width: 400px;
		margin-top: 60px;
	}

	.signup h2 {
		font-size: 2.4em;
	}

	.signup .field {
		margin-bottom: 16px;
	}
</style>

