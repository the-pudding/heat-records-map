<script>
	export let label;
	export let style = "inner";
	export let options = ["daily", "all-time"];
	export let value = options[0];

	let checked = value === options[0];

	const id = `toggle-${Math.floor(Math.random() * 1000000)}`;

	const translate = {
		"daily":"For any calendar day",
		"all-time":"All-time"
	}


	const handleClick = (event) => {
		const target = event.target;
		const state = target.getAttribute("aria-checked");
		checked = state === "true" ? false : true;
		value = checked ? options[0] : options[1];
	};
</script>

<div class="toggle toggle--{style}">
	<span class="label" {id}>{label}</span>
	<button
		role="switch"
		aria-checked={checked}
		aria-labelledby={id}
		on:click={handleClick}
	>
		{#if style === "inner"}
			<span>{translate[options[0]]}</span>
			<span style="">{translate[options[1]]}</span>
		{/if}
	</button>
</div>

<style>
	.toggle button,
	.label {
		font-family: inherit;
		font-size: 1em;
	}

	.toggle--inner [role="switch"][aria-checked="true"] :first-child,
	[role="switch"][aria-checked="false"] :last-child {
		display: inline-block;
		border-radius: 5px;
		padding: 8px 8px;
		background: #6e6e6e;
		font-weight: 500;
		color: #fff;
		-webkit-font-smoothing: antialiased;

	}

	.toggle--inner button {
		padding: 5px 5px;
		background: linear-gradient(45deg, rgb(248, 248, 248), rgb(232, 232, 232), rgb(248, 248, 248), rgb(232, 232, 232));
		border: 1px solid rgba(0,0,0,.06);
		border-radius: 5px;
	}


	.toggle--inner button span {
		user-select: none;
		pointer-events: none;
		display: inline-block;
		line-height: 1;
		font-weight: 500;
		color: rgb(76 76 76);
		padding: 0.25em;
		font-size: 15px;
	}

	.toggle--inner button:focus {
		box-shadow: 0 0 4px 0 var(--color-focus);
	}

	.toggle--slider {
		display: flex;
		align-items: center;
	}

	.toggle--slider button {
		width: 3.5em;
		height: 2em;
		position: relative;
		margin-left: 0.5em;
		background: var(--color-gray-300);
	}

	.toggle--slider button:focus {
		box-shadow: 0 0px 4px var(--color-focus);
	}

	.toggle--slider button::before {
		content: "";
		position: absolute;
		width: 1.5em;
		height: 1.5em;
		background: var(--color-white);
		border-radius: 4px;
		top: 0.25em;
		right: 1.75em;
	}

	.toggle--slider button[aria-checked="true"] {
		background-color: #6e6e6e;
		color: white;
		font-weight: 600;
	}

	.toggle--slider button[aria-checked="true"]::before {
		transform: translateX(1.5em);
	}

	.toggle--slider button:focus {
		box-shadow: 0 0 4px 0 var(--color-focus);
	}
</style>
