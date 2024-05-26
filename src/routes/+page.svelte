<script>
	// Import the functions you need from the SDKs you need
	import { getApps, getApp, deleteApp, initializeApp } from 'firebase/app';

	import {
		getFirestore,
		addDoc,
		collection,
		onSnapshot,
		query,
		orderBy,
		doc
	} from 'firebase/firestore';

	// https://firebase.google.com/docs/web/setup#available-libraries

	// Firebase configuration
	const firebaseConfig = {
		apiKey: 'AIzaSyCyFLOKW3Q7NZt4JJlzH3GZWd2gNlxnBuA',
		authDomain: 'artists-petition.firebaseapp.com',
		projectId: 'artists-petition',
		storageBucket: 'artists-petition.appspot.com',
		messagingSenderId: '181326708712',
		appId: '1:181326708712:web:08968fc9cecb9e5121ae5d',
		measurementId: 'G-4CWX96WJ3L'
	};

	// Initialize Firebase
	let firebaseApp;

	if (!getApps().length) {
		firebaseApp = initializeApp(firebaseConfig);
	} else {
		firebaseApp = getApp();
		deleteApp(firebaseApp);
		firebaseApp = initializeApp(firebaseConfig);
	}

	// Set up db & snapshot
	const db = getFirestore(firebaseApp);
	const orderedQuery = query(collection(db, 'votes'), orderBy('stamp', 'desc'));
	const unsubscribe = onSnapshot(orderedQuery, messageEvent);

	console.log(`It's alive`);

	const GOAL = 10;
	let votes = [];

	// Functions
	async function messageEvent(snapshot) {
		// Also init
		const docs = [];
		snapshot.docChanges().forEach(({ type, doc }) => {
			if (type === 'added') {
				docs.push(doc.data());
			} // } else if (type === 'removed') {
			// }
		});

		votes = [...docs, ...votes];
		console.log(votes);
	}

	async function postMessage(user = 'anonymous', txt = '👍') {
		const MAX_CHAR_LIMIT = 200;
		txt = txt.substring(0, MAX_CHAR_LIMIT);

		// Add document
		const obj = { user, msg: txt, stamp: new Date() };
		console.log(obj);
		await addDoc(collection(db, 'votes'), obj);
	}

	// UI handling such and such
	import { onMount } from 'svelte';
	const formatter = new Intl.DateTimeFormat('en-US', {
		dateStyle: 'long',
		timeStyle: 'short',
		hour12: true,
		formatRelativeTime: 'short'
	});

	let dialogElm;
	// let fx = [];

	onMount(() => {
		dialogElm.close();
	});

	function submitForm({ target: formElm }) {
		if (!formElm) return false;

		const formData = new FormData(formElm);
		const parsed = Object.fromEntries(formData.entries());
		postMessage(parsed.name, parsed.msg);

		formElm.reset();
		dialogElm.close();
	}
</script>

<svelte:window
	on:beforeunload={() => {
		console.log('Unsubscribed from event');
		unsubscribe();
	}}
/>

<svelte:head>
	<title>Artist Nickname Petition</title>
</svelte:head>

<img id="goal" src="/not-rigged-10.png" alt="" />

<h1>Petition</h1>
<h2>Artist nickname permissions petition :3</h2>

<button
	id="open-save-us"
	on:click={() => {
		dialogElm.showModal();
	}}>Join the Wall</button
>

<dialog bind:this={dialogElm}>
	<div id="other-stuff">
		<h3>Save us from our usernames 🙏</h3>
		<form id="main-thing" method="dialog">
			<input type="text" name="name" id="user-input" placeholder="alias" autocomplete="off" />
			<textarea name="msg" id="msg-input" placeholder="tell us more!"></textarea>
			<div>
				<input type="submit" value="Submit" on:submit|preventDefault={submitForm} />
				<button on:click={() => dialogElm.close()}>Close</button>
			</div>
		</form>
	</div>
</dialog>

<div id="stuff">
	<div>
		<h2>Count: {votes.length}/{GOAL}</h2>
	</div>
	<div id="stuff-container">
		{#each votes as { user, msg, stamp }, i}
			<div class="petition-user">
				<div>
					<h4 class="username">{user}</h4>
					<h4 class="timestamp">{formatter.format(stamp.toDate())}</h4>
				</div>
				<p>{msg}</p>
			</div>
		{/each}
	</div>
</div>

<div id="background-fx">
	<!-- {#each fx as { txt }, i}
		<h5 class="fx-fade">{txt}</h5>
	{/each} -->
</div>

<style>
	* {
		box-sizing: border-box;
	}

	h1 {
		font-size: 2.5em;
		margin-bottom: 5px;
		margin-top: 50px;
	}

	h2 {
		font-weight: lighter;
		margin-top: 5px;
		font-size: 1.2em;
	}

	h3 {
		font-size: 1.5em;
		margin: 5px 0px;
		margin-bottom: 15px;
	}

	h4 {
		font-weight: lighter;
		margin: 2px;
		color: rgb(165, 165, 165);
	}

	input,
	textarea {
		outline: none;
		border-width: 0;
	}

	dialog[open] {
		background-color: rgb(11, 11, 11, 0);
		display: flex;
		align-items: center;
		flex-direction: column;
		color: white;
		border: none;
		border-radius: 10px;
		width: 75%;
	}

	dialog::backdrop {
		background-color: rgba(0, 0, 0, 0.8);
	}

	button {
		cursor: pointer;
		border: none;
		border-radius: 8px;
		background-color: rgb(43, 43, 43);
		color: white;
		padding: 10px 15px;
		font-size: 1.1em;
		margin: 15px;
		transition: 0.4s ease-out background-color;
		outline: none;
	}

	button:hover {
		background-color: rgb(43, 43, 43, 0.5);
	}

	#goal {
		width: 100%;
		height: auto;
		max-width: 100%;
		object-fit: contain;
		opacity: 0.2;

		z-index: -5;
	}

	/* Votes display (for the most part at least) */
	.petition-user {
		display: flex;
		align-items: flex-start;
		flex-direction: column;
		padding: 20px;
		border-radius: 8px;

		width: 75%;
		margin: 5px 0px;

		background-color: rgb(2, 2, 2);
	}

	.petition-user > p {
		margin: 1px;
	}

	.username {
		width: 100%;
		text-align: left;
		font-size: 1.12em;
		color: white;
	}

	.timestamp {
		font-size: 0.8em;
		width: 100%;
		text-align: left;
	}

	.petition-user > div {
		margin-bottom: 7px;
	}

	/* Main votes div */
	#stuff {
		display: flex;
		align-items: center;
		flex-direction: column;
		width: 100%;
		margin-top: 50px;
	}

	#stuff-container {
		display: flex;
		align-items: center;
		flex-direction: column;
		width: 100%;
	}

	/* Other stuff */
	#other-stuff {
		width: 70%;
		display: flex;
		align-items: center;
		flex-direction: column;
		margin-top: 25px;
	}

	/* #main-thing = form element */
	#main-thing {
		display: flex;
		align-items: center;
		flex-direction: column;
		gap: 15px;
		width: 100%;
	}

	#msg-input,
	#user-input {
		width: 80%;
		background-color: rgb(34, 34, 34);
		color: white;
		padding: 10px;
		font-size: 1em;
		border-radius: 8px;
	}

	#user-input {
		/* width: 50%; */
		padding: 10px;
		background-color: rgb(22, 22, 22);
	}

	#msg-input {
		height: auto;
		resize: vertical;
	}

	input[type='submit'] {
		border: none;
		padding: 10px 20px;
		color: rgb(255, 255, 255);
		border-radius: 6px;
		font-size: 0.95em;
		margin: 3px 5px;
		cursor: pointer;

		background-color: rgb(65, 147, 255);
		transition: 0.25s ease-out background-color;
	}

	input[type='submit']:hover {
		background-color: rgb(65, 147, 255, 0.5);
	}

	form div {
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: row;
	}

	form div button {
		font-size: 0.95em;
		padding: 10px 20px;
	}

	form *::placeholder {
		color: rgba(255, 255, 255, 0.5);
	}

	@media only screen and (max-width: 600px) {
		/* Styles for small devices (e.g., smartphones) */
		.petition-user {
			width: 75%;
		}

		#other-stuff {
			width: 100%;
		}

		dialog[open] {
			width: 100%;
		}
	}

	@media only screen and (min-width: 601px) and (max-width: 1160px) {
		/* Styles for medium devices (e.g., tablets) */
		.petition-user {
			width: 50%;
		}

		#other-stuff {
			width: 65%;
		}

		#goal {
			width: 75%;
		}
	}

	@media only screen and (min-width: 1161px) {
		/* Styles for large devices (e.g., desktops and laptops) */
		.petition-user {
			width: 33%;
		}

		#other-stuff {
			width: 33%;
		}

		#goal {
			height: 50px;
		}
	}
</style>
