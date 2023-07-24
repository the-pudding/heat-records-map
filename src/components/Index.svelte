<script>
	import { onMount } from "svelte";
	import { csv, minIndex } from "d3";
	import mapboxgl from 'mapbox-gl'; // or "const mapboxgl = require('mapbox-gl');"

	// import Map from "$components/Map.svelte"
	let url = "https://pudding.cool/2022/03/weather-map-data/acis.csv"
	let data = null;
    let dataType = "daily"
	let least;
	let leastIndex;
	let innerHeight;



	onMount(async () => {
		csv(url).then(function(response) {

			leastIndex = minIndex(response, d => {
				if(dataType == "daily"){
					return +d["days_since_daily"]
				}
				return +d["all_time_days"];
			});
  
    		least = response[leastIndex];


			data = response;
		});
	})
</script>


<div class="top">
	<div class="white-fade">
	</div>
	<h1>Days Since a Record-high Temperature for a Particular Day</h1>
</div>

<div class="bottom">
	<div class="white-fade">
	</div>
	<h3>A Project from <a target="_blank" href="/">The Pudding</a></h3>
</div>


{#if data}
	<div class="map-wrapper"
		style="
			height: {innerHeight}px;
		"
	>
		<!-- <Map lat={35} lon={-84} zoom={3.5} data={data} least={least} timeframe={dataType}>
		</Map> -->
	</div>
{/if}

<div class="toolbar">
</div>


<svelte:window bind:innerHeight />

<style>

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
		backdrop-filter: blur(100px) saturate(400%) brightness(80%);
		height: 60px;
		z-index: 100000;
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

	.bottom .white-fade {
		width: 100%;
		height: 100%;
		position: absolute;
		background: linear-gradient(0deg, #FFF 0%, rgba(255,255,255,0) 100%);
		z-index: -1;
	}
	.map-wrapper {
		height: 100%;
		width: 100%;
	}
</style>