<script>
	import { onMount } from "svelte";

	/* 
		https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
		https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials
		https://developer.mozilla.org/en-US/docs/Web/API/Request/mode
		https://flask-cors.readthedocs.io/en/latest/api.html#using-cors-with-cookies

	*/
	/*
	async function postData(url = "", data = {}) {
		// Default options are marked with *
		const response = await fetch(url, {
			method: "POST", // *GET, POST, PUT, DELETE, etc.
			mode: "cors", // no-cors, *cors, same-origin
			cache: "no-cache", // *default, no-cache, reload, force-cache, only-if-cached
			credentials: "same-origin", // include, *same-origin, omit
			headers: {
				"Content-Type": "application/json",
				// 'Content-Type': 'application/x-www-form-urlencoded',
			},
			redirect: "follow", // manual, *follow, error
			referrerPolicy: "no-referrer", // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
			body: JSON.stringify(data), // body data type must match "Content-Type" header
		});
		return response.json(); // parses JSON response into native JavaScript objects
	}
	*/

	let BASE_URL =
		"https://some-fancy-name-at.cfapps.eu10.hana.ondemand.com/alarmed-id/";

	let alarm_id_current_items = [];
	let alarm_id_removed_items = [];
	let alarm_id_to_add = "";
	let data;

	async function drop_alarm_id(item) {
		let alarm_id_url = BASE_URL + item;
		let args = {
			credentials: "include",
			mode: "cors",
			method: "DELETE",
		};
		data = await fetch(alarm_id_url, args).then((x) => x.json());
		alarm_id_current_items = data["index"];
		alarm_id_removed_items = alarm_id_removed_items.concat([item]).sort();
	}

	async function add_alarm_id(item) {
		let alarm_id_url = BASE_URL + item;
		let args = {
			credentials: "include",
			mode: "cors",
			method: "PUT",
		};
		data = await fetch(alarm_id_url, args).then((x) => x.json());
		alarm_id_current_items = data["index"];
		alarm_id_to_add = "";
		alarm_id_removed_items = alarm_id_removed_items
			.filter((x) => x !== item)
			.sort();
	}

	onMount(async () => {
		let args = {
			credentials: "include",
			mode: "cors",
		};
		data = await fetch(BASE_URL, args).then((x) => x.json());
		alarm_id_current_items = data["index"].sort();
	});

	function handle_request_to_add() {
		console.info(`alarm_id_to_add: ${alarm_id_to_add}`);

		try {
			let mangled = alarm_id_to_add.trim();
			if (mangled !== "") {
				add_alarm_id(mangled);
			} else {
				console.warn("yeah, right.");
			}
		} catch (e) {
			console.warn("Schubiduh ..");
		}
	}
</script>

<main>
	<h1>Alarmidas</h1>
	<em>{BASE_URL}</em>
	<div>
		<form on:submit|preventDefault={handle_request_to_add}>
			<input
				type="text"
				bind:value={alarm_id_to_add}
				placeholder="0711"
			/>
			<button on:click={handle_request_to_add}>Add ID</button>
		</form>
	</div>

	<h2>Current Alarm IDs</h2>
	{#if alarm_id_current_items.length == 0}
		<p>No items?! Hurry, add some!</p>
	{/if}

	{#each alarm_id_current_items as item}
		<button
			title="remove {item}"
			class="remove"
			on:click={() => drop_alarm_id(item)}>❌ {item}</button
		>
	{/each}

	<h2>Removed Alarm IDs</h2>
	{#if alarm_id_removed_items.length == 0}
		<p>No items removed (yet?).</p>
	{/if}

	{#each alarm_id_removed_items as item}
		<button
			title="restore {item}"
			class="restore"
			on:click={() => add_alarm_id(item)}>+ {item}</button
		>
	{/each}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	button {
		min-width: 6rem;
		padding: 0.4rem;
		margin-left: 0.5rem;
		margin-right: 0.5rem;
	}

	button.remove {
		background-color: rgba(170, 0, 0, 0.1);
		border-color: rgba(170, 0, 0, 0.5);
	}

	button.restore {
		background-color: rgba(0, 170, 0, 0.1);
		border-color: rgba(0, 170, 0, 0.5);
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
