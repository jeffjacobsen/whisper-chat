<template>
	<div>
		<h1>Whisper Chat</h1>
		<div v-if="!configured">
			Symmetric Password: <input v-model="sympw" @input="updateSymKey(sympw)" /><br><br>
			Username: <input v-model="name" /><br><br>
			<button @click="configWithKey" v-if="symKeyId && name">Start</button>
		</div>
		<div v-else>
			<div class="columns">
				<div>
					<p v-for="m of msgs">
						<b>{{m.name}}</b>: {{m.text}}
					</p>
					Message: <input v-model="text" @keyup.enter="sendMessage" />
					<button @click="sendMessage">Send</button>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import Web3 from 'web3';
import {decodeFromHex, encodeToHex} from './hexutils';

export default {
	data() {
		this.web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8546"));
		this.shh = this.web3.shh;

		let data = {
			msgs: [],
			sent: [],
			text: "",
			symKeyId: null,
			name: "",
			sympw: "",
			configured: false,
			topic: null,
			symKey: ""
		};

		/* generate private keypair
		this.shh.newKeyPair().then(id => {
			data.asymKeyId = id;
			return this.shh.getPublicKey(id).then(pubKey => this.asymPubKey = pubKey).catch(console.log);
		}).catch(console.log);
		*/

		return data;
	},

	methods: {
		sendMessage() {
			let msg = {
				text: this.text,
				name: this.name
			};

			let postData = {
			    symKeyID: this.symKeyId,
                topic: this.topic,
                payload: encodeToHex(JSON.stringify(msg)),
				ttl: 7,
				powTarget: 2.01,
				powTime: 100
			};

			this.shh.post(postData);
			this.text = "";
            // this.sent.push(msg);
		},

        updateSymKey(sympw) {
			this.shh.generateSymKeyFromPassword(sympw).then(symKeyID => {
				this.shh.getSymKey(symKeyID).then(symKey => {
					this.topic = symKey.substring(0, 10); // use first 8 hex characters from key as topic
					this.symKey = symKey
					this.symKeyId = symKeyID
				})
			})

		},

		configWithKey() {
			if (!this.name || this.name.length === 0) {
				alert("Please pick a username");
				return;
			}
            if (!this.symKeyId || this.symKeyId.length === 0) {
                alert("please enter a password to generate a key!");
                return;
            }

			let filter = {
				topics: [this.topic],
				symKeyID: this.symKeyId
			};
			this.shh.subscribe('messages', filter, (error, notification) => {
				let message = decodeFromHex(notification.payload);
				this.msgs.push({
					name: message.name,
					text: message.text
				});
			})
			this.configured = true;
		}
	}
};
</script>

<style>
	.columns {
		display: flex;
	}
	.columns > div {
		flex: 1;
	}
</style>
