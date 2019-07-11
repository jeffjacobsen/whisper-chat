<template>
	<div>
		<h1>Whisper Chat</h1>
		<div v-if="!configured">
			Topic Password: <input v-model="sympw" @input="updateSymKey(sympw)" /><br><br>
			Username: <input v-model="name" /><br><br>
			<button @click="configWithKey" v-if="symKeyId && name">Start</button>
		</div>
		<div v-else>
			<div>
				Topic: {{topic}}<br>
				Key: {{symKey}}
			</div>
			<div class="columns">
				<div>
					<p v-for="m of msgs">
						<b>{{m.name}}</b>: {{m.text}}
					</p>
				</div>
				<div>
					<p v-if="sent.length">SENT by {{name}}:</p>
					<p v-for="m of sent">
						<b>></b> {{m.text}}
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
		this.web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
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

		this.shh.newKeyPair().then(id => {
			data.asymKeyId = id;
			return this.shh.getPublicKey(id).then(pubKey => this.asymPubKey = pubKey).catch(console.log);
		}).catch(console.log);

		return data;
	},

	methods: {
		sendMessage() {
			let msg = {
				text: this.text,
				name: this.name
			};

			this.sent.push(msg);

			let postData = {
				ttl: 7,
				topic: this.topic,
				powTarget: 2.01,
				powTime: 100,
				payload: encodeToHex(JSON.stringify(msg)),
			};

			postData.symKeyID = this.symKeyId;

			this.shh.post(postData);

			this.text = "";
		},

        updateSymKey(sympw) {
            Promise.all([
                this.shh.generateSymKeyFromPassword(sympw).then(symKeyID => this.symKeyId = symKeyID),
            ]).then(() => {
                this.shh.getSymKey(this.symKeyId).then(symKey => {
                    this.topic = symKey.substring(0, 10); // use first 8 hex characters from key as topic
                    this.symKey = symKey
                })
            })
        },

		configWithKey() {
			// TODO use a form
			if (!this.name || this.name.length == 0) {
				alert("Please pick a username");
				return;
			}

			let filter = {
				topics: [this.topic]
			};


			if (!this.symKeyId || this.symKeyId.length == 0) {
				alert("please enter a pasword to generate a key!");
				return;
			}

			filter.symKeyID = this.symKeyId

			this.msgFilter = this.shh.newMessageFilter(filter).then(filterId => {
				setInterval(() => {
					this.shh.getFilterMessages(filterId).then(messages => {
						for (let msg of messages) {
							let message = decodeFromHex(msg.payload);
							this.msgs.push({
								name: message.name,
								text: message.text
							});
						}
					});
				}, 1000);
			});

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
