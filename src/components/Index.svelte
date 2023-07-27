<script>
	import { onMount } from "svelte";
	import { csv, minIndex } from "d3";
	import { fly, fade } from "svelte/transition"
	import Toggle from "$components/helpers/Toggle.svelte";
	import Demo from "$components/demo/Demo.svelte"
	import { List, Globe2 } from "lucide-svelte";

	import ListView from "$components/ListView.svelte"

	// import mapboxgl from 'mapbox-gl'; // or "const mapboxgl = require('mapbox-gl');"

	import Map from "$components/Map.svelte"
	let url = "https://pudding.cool/2022/03/weather-map-data/acis.csv"
	let data;
    let dataType = "daily"
	let least;
	let leastIndex;
	let innerWidth;
	let innerHeight;
	let moveTitles;

	let projectView = "map";

	let valueSlider = "daily";

	let toolbarDelay = 0;//6000;


	let toRemove = ["Charleston Downtown Area", "Austin-Bergstrom Airport Area", "Portland Downtown Area"]

	const switchDataType = () => {
		console.log("switching datatype",dataType)
		if(dataType == "daily"){
			dataType = "all-time"
		}
		else {
			dataType = "daily"
		}
	}

	const mapboxLoaded = () => {
		console.log("here")
		moveTitles = true;
	}

	const changeView = () => {
		if(projectView == "map"){
			projectView = "list";
		}
		else {
			projectView = "map";
		}
	}

	async function setLeast(response){

		console.log("setting least", dataType)

		leastIndex = minIndex(response, d => {
				if(dataType == "daily"){
					return +d["days_since_daily"]
				}
				return +d["all_time_days"];
			});
  
    	least = response[leastIndex];

		return;
	}


	onMount(async () => {
		csv(url).then(async function(response) {

			await setLeast(response);

			data = response.filter(d => {
				return toRemove.indexOf(d.name) == -1;
			});
		});
	})

	$: dataType, data ? setLeast(data) : '';

	$: console.log(valueSlider)
	$: dataType = valueSlider;
</script>


<!-- <Demo /> -->

<div class="bottom">
	<div class="white-fade not-loaded" class:moveTitles>
	</div>
	<h3>A Project from <a target="_blank" href="/">The Pudding</a></h3>
</div>

<div class="top">
	<div class="white-fade not-loaded" class:moveTitles>
	</div>
</div>

{#if data}
<div class:moveTitles in:fade={{delay:1000, duration:1000}} class="title">
	{#if dataType == "daily"}
		<h1 out:fly={{y:-20, duration:1000}}>Days since a city had a record-high temperature</h1>
	{:else}
		<h1 in:fly={{y:20, duration:1000, delay:1000}}>Time since a city had the highest temperature ever recorded</h1>
	{/if}
</div>
{/if}

{#if !data}
	<div out:fly={{delay:500, y:-20, duration:500}} class="loading">
		<p>loading...</p>
	</div>
{/if}

{#if data}

	{#if projectView == "list"}
		<div class="list" transition:fly={{x:innerWidth,duration:500, delay:500, opacity:1}}>
			<div class="inner-list">
				<ListView {data} {dataType} />
			</div>
		</div>
	{/if}

	<div in:fade={{delay:2000}} class="map-wrapper"
		style="
			height: {innerHeight}px;
		"
	>
		<Map on:loaded={mapboxLoaded} lat={least.true_latitude} lon={least.true_longitude} zoom={1} data={data} least={least} timeframe={dataType}>
		</Map>
	</div>
{/if}

{#if moveTitles}


	<div in:fly={{y:20, duration:1000, delay:toolbarDelay}} class="toolbar">

		<section class="timeframe-toggle">
			<!-- <h2>Toggle (inner) <span>{valueInner}</span></h2>
			<Toggle label="Enable" style="inner" bind:value={valueInner} /> -->
			<p>Time since a heat record was set...</p>
				<Toggle label="" style="inner" bind:value={valueSlider} />
		</section>
		<div class="project-view-toggle">
			<p>{projectView == "map" ? "List" : "Map"} View</p>
			<button on:click={() => changeView()} class="toolbar-button">
				{#if projectView == "map"}
					<List color={"#000"} strokeWidth={2} />
				{:else}
					<Globe2 color={"#000"} strokeWidth={2} />
				{/if}
				
			</button>
	
		</div>
		
		<!-- <button aria-label="GitHub" class="c-gasElL" data-state="tooltip-hidden" data-reach-tooltip-trigger="" style="width: 40px; height: 40px; top: 0px;"><div aria-hidden="true" class="c-lesPJm c-lesPJm-igHjMGo-css"></div><svg viewBox="0 0 24 24" width="24" height="24" fill="currentColor"><path fill="none" d="M0 0h24v24H0z"></path><path d="M12 2C6.475 2 2 6.475 2 12a9.994 9.994 0 0 0 6.838 9.488c.5.087.687-.213.687-.476 0-.237-.013-1.024-.013-1.862-2.512.463-3.162-.612-3.362-1.175-.113-.288-.6-1.175-1.025-1.413-.35-.187-.85-.65-.013-.662.788-.013 1.35.725 1.538 1.025.9 1.512 2.338 1.087 2.912.825.088-.65.35-1.087.638-1.337-2.225-.25-4.55-1.113-4.55-4.938 0-1.088.387-1.987 1.025-2.688-.1-.25-.45-1.275.1-2.65 0 0 .837-.262 2.75 1.026a9.28 9.28 0 0 1 2.5-.338c.85 0 1.7.112 2.5.337 1.912-1.3 2.75-1.024 2.75-1.024.55 1.375.2 2.4.1 2.65.637.7 1.025 1.587 1.025 2.687 0 3.838-2.337 4.688-4.562 4.938.362.312.675.912.675 1.85 0 1.337-.013 2.412-.013 2.75 0 .262.188.574.688.474A10.016 10.016 0 0 0 22 12c0-5.525-4.475-10-10-10z"></path></svg></button> -->

		<!-- <button on:click={() => switchDataType()}>all-time</button> -->
	</div>
{/if}


<svelte:window bind:innerHeight bind:innerWidth />

<style>
	.list {
		position: absolute;
		top: 0;
		left: 20px;
		right: 0;
		width: calc(100% - 20px);
		z-index: 10000;
		height: 100%;
		overflow: scroll;
		background-color: white;
		border-left: 1px solid #ddd;

		border-left: 1px solid rgba(0, 0, 0, 0.07);
		box-shadow: -20px 30px 60px rgba(0, 0, 0, 0.12);
		background: rgba(255, 255, 255, 0.5);
		backdrop-filter: blur(100px) saturate(400%) brightness(100%);

	}
	.inner-list {
		width: calc(100% - 50px);
		margin: 0 auto;
		max-width: 400px;
	}
	.timeframe-toggle, .project-view-toggle {
		display: flex;
		align-self: center;
	}

	.project-view-toggle p {
		text-transform: capitalize;
	}

	.timeframe-toggle p, .project-view-toggle p {
		margin: 0;
		align-self: center;
		font-size: 14px;
		font-weight: 500;
		max-width: 140px;
		text-align: right;
		line-height: 1.2;
		margin-right: 13px;
	}
	.title {
		position: absolute;
		top: 50%;
		transform: translate(0,-50%);
		z-index: 10000;
		width: 100%;
	}

	.loading {
		font-family: var(--sans);
		position: absolute;
		top: 50%;
		left: 0;
		right: 0;
		margin: 0 auto;
		transform: translate(0,-50%);
		text-align: center;
		z-index: 1000000;
	}
	.loading p {
		font-size: 36px;
	}

	.not-loaded {
		opacity: 0;
	}

	.toolbar {
		position: fixed;
		bottom: 0;
		transform: translate(0, calc(-50% - 10px));
		width: calc(100% - 50px);
		left: 0;
		right: 0;
		margin: 0 auto;
		border-radius: 9999px;
		border: 1px solid rgba(0, 0, 0, 0.07);
		box-shadow: 0 30px 60px rgba(0, 0, 0, 0.12);
		background: rgba(255, 255, 255, 0.7);
		backdrop-filter: blur(100px) saturate(400%) brightness(100%);
		height: 60px;
		z-index: 100000;
		display: flex;
		justify-content: space-between;
		padding: 0 10px;


		background: rgba(255, 255, 255, 0.9);
		backdrop-filter: blur(100px) saturate(400%) brightness(90%);
	}

	h1, h3 {
		font-size: 18px;
		text-align: center;
		margin: 0 auto;
		width: calc(100% - 50px);
		margin-top: 10px;
	}
	h3 {
		font-size: 12px;
		margin: 0 auto;
		margin-bottom: 10px;
	}
	.top {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		margin: 0 auto;
		z-index: 1000;
		height: 100px;
	}

	.bottom {
		bottom: 0;
		position: fixed;
		z-index: 10000;
		width: 100%;
		height: 100px;
		display: flex;
		flex-direction: column;
		justify-content: flex-end;
	}

	.bottom .white-fade, .top .white-fade {
		width: 100%;
		height: 100%;
		position: absolute;
		background: linear-gradient(0deg, #FFF 0%, rgba(255,255,255,0) 100%);
		z-index: -1;
	}

	.top .white-fade {
		background: linear-gradient(180deg, #FFF 20%, rgba(255,255,255,0) 100%);
	}
	.map-wrapper {
		height: 100%;
		width: 100%;
	}

	.moveTitles.title {
		top: 0;
		transform: translate(0,0%);
		transition: top 1s, transform 1s;
	}

	.moveTitles.not-loaded {
		opacity: 1;
	}

	.toolbar .toolbar-button {
		background: linear-gradient(45deg, rgb(248, 248, 248), rgb(232, 232, 232), rgb(248, 248, 248), rgb(232, 232, 232));
		border-radius: 100%;
		height: 43px;
		align-self: center;
		width: 45px;
		border: 1px solid rgba(0,0,0,.06);
	}
	
</style>