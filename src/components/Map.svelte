<script>
	import { onDestroy } from 'svelte';
    // import mapboxgl from 'mapbox-gl'; // or "const mapboxgl = require('mapbox-gl');"
    import { onMount } from "svelte";
    import { scaleLinear, selectAll, timeFormat, geoAlbersUsa, geoMercator } from "d3";
    import viewport from "$stores/viewport.js";

    export let data;
    export let least;
    export let timeframe;

    mapboxgl.accessToken = 'pk.eyJ1IjoiZG9jazQyNDIiLCJhIjoiY2xrZ3llNmFkMDRpYjNkcDRuNDQ3MDBmayJ9.r7nPJo7xLNkUAYWp7QO9_g';
	// set the context here...

		export let lat;
		export let lon;
		export let zoom;

    const formatTime = timeFormat("%B %d, %Y");
    const formatTimeNoYear = timeFormat("%B %d");

    let popupDataToPass = least;
    let popupString = "hi"

    $: w = $viewport.width;
    $: h = $viewport.height;
    // $: popupDataDate = new Date(popupData.days_since_daily_date.slice(0,4), +popupData.days_since_daily_date.slice(5,7) -1, popupData.days_since_daily_date.slice(8,10));

    let layers = [
        "daily","daily-days","monthly","alltime"
    ]

    let layersMin = [
        "daily-night","daily-days-night","monthly-night","alltime-night"
    ]

    function popupCreate(dataToMake){

        let first = `The <b>${dataToMake.name.replace(" Area","")}, ${dataToMake.state_abbr} area</b> broke the `;
        let second = `${timeframe} record high <b>`
        let third = Math.floor(dataToMake.all_time_days/365);
        let fourth = " years ";
        if(third == 1){
            fourth = " year ";
        }

        let fifth = dataToMake.all_time_days % 365 + " days"
        if(third == 0) {
            third = dataToMake.all_time_days
            fourth = " days"
            fifth = ""
        }

        if(timeframe == "daily"){
            third = dataToMake.days_since_daily;
            fourth = " days";
            if(third == 1){
                fourth = "day"
            }
            fifth = "";
        }

        let sixth = ` ago</b> with a temperature of `
        let dateNoSuffix = new Date(dataToMake.all_time_date.slice(0,4), +dataToMake.all_time_date.slice(5,7) -1, dataToMake.all_time_date.slice(8,10));

        if(timeframe == "daily"){
            dateNoSuffix = new Date(dataToMake.days_since_daily_date.slice(0,4), +dataToMake.days_since_daily_date.slice(5,7) -1, dataToMake.days_since_daily_date.slice(8,10));
        }

        let seventh = `${dataToMake.all_time_val}°F. That&rsquo;s the hottest temperature on record for the area.`;

        if(timeframe == "daily"){
            seventh = `${dataToMake.days_since_val}°F. That&rsquo;s the hottest ${formatTimeNoYear(dateNoSuffix)} on record for the area.`;
        }


        return first + second + third + fourth + fifth + sixth + seventh;
    }

    let mounted = false;
    $: if(mounted == true) load(least);

    onMount(async () => {
        mounted = true;
    });

	let container;
	let map;

    let dataMap = new Map(data.map(d => [d.station,d]))

    const bounds = [
            [-22.275411,-13.090124],
            [20.68255,12.703398]
    ];

    const maxBounds = [
            [-30.275411,-20.090124],
            [27.68255,20.703398]
    ];


    let center = [-0.7964304999999996,-0.19336299999999973];

    let stationIds = data.map(function(d){
        return d.station;
    })

    let popup = null;
    let padding = {top:50};
    let navPosition = 'top-right';
    let scrollZoom = true;
    let initialZoom = 4;

	function load(dataPoint) {
        if(w < 500) {
            padding = 0;
            navPosition = 'bottom-right';
            // scrollZoom = true;
            // initialZoom = 3;
        }

        let geojson = {
                "type":"FeatureCollection",
                "features":null
        };

        geojson.features = data.map(d => {
            return {
                "type":"Feature",
                "properties":{"station":d.station},
                "id":stationIds.indexOf(d.station),
                "geometry":{
                    "type":"Point",
                    "coordinates":[d.longitude,d.latitude]
                }
            }
        })

        if(!map){


            map = new mapboxgl.Map({
                container,
                style: 'mapbox://styles/dock4242/cl0bhzae7000y14utgxgy71f1', //'mapbox://styles/dock4242/cl0banu3w000i14l8w1woe65r',
                maxBounds: maxBounds,
                center: center,//[-95.973515, 38.382024],//,
                zoom: initialZoom,
                scrollZoom: scrollZoom
		    });
        }
        else {
            map.remove();
            map = new mapboxgl.Map({
                container,
                style: 'mapbox://styles/dock4242/cl0bhzae7000y14utgxgy71f1', //'mapbox://styles/dock4242/cl0banu3w000i14l8w1woe65r',
                maxBounds: maxBounds,
                center: center,//[-95.973515, 38.382024],//,
                zoom: initialZoom,
                scrollZoom: scrollZoom
		    });
        }

        let popupAdded = false;
        let hoveredStation = null;

        map.on('load', function(){

            map.fitBounds(bounds, {padding: padding});

            let sizeControlLength = selectAll(".mapboxgl-ctrl-zoom-in").size();

            if(sizeControlLength == 0){
                // map.addControl(new mapbox.AttributionControl(), 'top-left');
                // map.addControl(new mapbox.NavigationControl({visualizePitch:false, showCompass: false}), navPosition);
            }

            if(w < 500){
                // map.scrollZoom.disable();
            }

            function sizeFont(days){

                if(timeframe == "all-time"){
                    if(w < 500){
                        if(days < 365){
                            return 16
                        }
                        if(days < 3650){
                            return 14
                        }
                        return 11;
                    }

                    if(days < 365){
                        return 20
                    }
                    if(days < 365 * 10){
                        return 16
                    }
                    return 12;
                }


                if(w < 500){
                    if(days < 30){
                        return 16
                    }
                    if(days < 60){
                        return 14
                    }
                    return 11;
                }

                if(days < 30){
                    return 20
                }
                if(days < 60){
                    return 16
                }
                return 14;
            }

            let sources = map.getStyle().sources
            if(Object.keys(sources).indexOf("test-circles") == -1){
                map.addSource('test-circles', {
                    'type': 'geojson',
                    'data': geojson
                });

                let opacity = 1;
                if(timeframe == "all-time"){
                    opacity = .25;
                }

                map.addLayer({
                    'id': 'hover-effect',
                    'type': 'circle',
                    'source': 'test-circles',
                    'layout': {},
                    'paint': {
                        'circle-stroke-color': '#000',
                        'circle-radius': 20,
                        'circle-stroke-width': 2,
                        'circle-color': "rgba(0,0,0,0)",
                        'circle-stroke-opacity':[
                            'case',
                            ['boolean', ['feature-state', 'hover'], false],
                            opacity,
                            0
                        ]
                    }
                });
            }

            const matchExpression = ['match', ['get','station']];
            const sizeExpression = ['match', ['get','station']];
            const colorExpression = ['match', ['get','station']];
            let colorScale = scaleLinear().domain([10000,100]).range(["#585656","#030202"]).clamp(true);

            for (const row of data){
                if(timeframe == "daily"){
                    matchExpression.push(row['station'],row["days_since_daily"]);
                    sizeExpression.push(row['station'],sizeFont(+row["days_since_daily"]));
                    colorExpression.push(row['station'],"#000000");
                }
                else if(timeframe == "all-time"){
                    let textForCell = Math.floor(row["all_time_days"]/365) + " yrs."

                    if(Math.floor(row["all_time_days"]/365) == 1){
                        textForCell = Math.floor(row["all_time_days"]/365) + " yr."
                    }

                    if(Math.floor(row["all_time_days"]/365) == 0) {
                        textForCell = row["all_time_days"] + " days"
                    }

                    if(row["station"] == least["station"]){
                        textForCell = row["all_time_days"];
                    }

                    matchExpression.push(row['station'],textForCell);
                    sizeExpression.push(row['station'],sizeFont(+row["all_time_days"]));
                    colorExpression.push(row['station'],colorScale(+row["all_time_days"]));
                }
            }

            matchExpression.push("red");
            colorExpression.push("red");
            sizeExpression.push(12);

						let dailyThreshold = 30;

            let stationsMid = data.filter(function(d){
                if(timeframe == "all-time"){
                    return +d["all_time_days"] < 1000 && d["station"] != least.station;
                }
                return +d["days_since_daily"] < dailyThreshold && d["station"] != least.station;
            }).map(d => d.station);

						if(stationsMid.length == 0){
							dailyThreshold = 100;

							stationsMid = data.filter(function(d){
	                if(timeframe == "all-time"){
	                    return +d["all_time_days"] < 1000 && d["station"] != least.station;
	                }
	                return +d["days_since_daily"] < dailyThreshold && d["station"] != least.station;
	            }).map(d => d.station);
						}

            let stationsBase = data.filter(function(d){
                if(timeframe == "all-time"){
                    return +d["all_time_days"] > 999 && d["station"] != least.station;
                }
                return +d["days_since_daily"] > (dailyThreshold - 1) && d["station"] != least.station;
            }).map(d => d.station);




            map.setPaintProperty('acis-reproj-7ytdey-mid-layer', 'text-color', colorExpression);
            map.setPaintProperty('acis-reproj-7ytdey', 'text-color', colorExpression);

            map.setPaintProperty('state-boundaries copy', 'fill-color', "#e8e8e8");

            map.setLayoutProperty('acis-reproj-7ytdey-top-layer', 'text-field', matchExpression);
            map.setLayoutProperty('acis-reproj-7ytdey-top-layer-mobile', 'text-field', matchExpression);
            map.setLayoutProperty('acis-reproj-7ytdey-mid-layer', 'text-field', matchExpression);
            map.setLayoutProperty('acis-reproj-7ytdey', 'text-field', matchExpression);

            if(w < 500) {
                map.setLayoutProperty('acis-reproj-7ytdey-top-layer', 'visibility', "none");
                map.setLayoutProperty('acis-reproj-7ytdey-top-layer-mobile', 'visibility', "visible");
            }
            else {
                map.setLayoutProperty('acis-reproj-7ytdey-top-layer', 'visibility', "visible");
                map.setLayoutProperty('acis-reproj-7ytdey-top-layer-mobile', 'visibility', "none");
            }

            map.setLayoutProperty('acis-reproj-7ytdey-mid-layer', 'text-size', sizeExpression);
            map.setLayoutProperty('acis-reproj-7ytdey', 'text-size', sizeExpression);

            map.setFilter('acis-reproj-7ytdey-top-layer', [ "all", [ "match", ["get", "station"], [least.station], true, false ] ]);
            map.setFilter('acis-reproj-7ytdey-top-layer-mobile', [ "all", [ "match", ["get", "station"], [least.station], true, false ] ]);
            map.setFilter('acis-reproj-7ytdey-mid-layer', [ "all", [ "match", ["get", "station"], stationsMid, true, false ] ]);
            map.setFilter('acis-reproj-7ytdey', [ "all", [ "match", ["get", "station"], stationsBase, true, false ] ]);


            if(!popupAdded) {

                popup = new mapboxgl.Popup({ focusAfterOpen: false, maxWidth: '340px' });
                popup.addClassName('last-record');

                // popupString = `The <b>${least.name.replace(" Area","")}, ${least.state_abbr} area</b> broke the all-time high <b>${least.all_time_days} days ago</b> with a temperature of ${least.days_since_val}°F. That&rsquo;s the hottest ${formatTimeNoYear(leastDate)} on record for the area.`;

                // if(timeframe == "all-time"){
                //     popupString = `The <b>${least.name.replace(" Area","")}, ${least.state_abbr} area</b> broke the daily record high <b>${+least.all_time_days > 365 ? Math.floor(least.all_time_days/365) + " years " + least.all_time_days % 365 + " days" : least.all_time_days + " days"} ago</b> with a temperature of ${least.days_since_val}°F. That&rsquo;s the hottest ${formatTimeNoYear(leastDate)} on record for the area.`;
                // }

                popupDataToPass = least;

                popup
                    .setLngLat([least.longitude,least.latitude])
                    //.setHTML("hi")
                    .setHTML(popupCreate(popupDataToPass))
                    .addTo(map);

                popupAdded = true;
            }

            map.on('mousemove', (e) => {

                const features = map.queryRenderedFeatures(e.point);
                // Limit the number of properties we're displaying for
                // legibility and performance
                const displayProperties = [
                    'type',
                    'properties',
                    'id',
                    'layer',
                    'source',
                    'sourceLayer',
                    'state'
                ];

                const displayFeatures = features.map((feat) => {
                    const displayFeat = {};
                    displayProperties.forEach((prop) => {
                        displayFeat[prop] = feat[prop];
                    });
                    return displayFeat;
                }).filter(function(d){
                    return d.layer.id.includes("acis")
                });

                if(displayFeatures.length > 0 && popupAdded){

                    let popupData = displayFeatures[0].properties;

                    if(hoveredStation !== stationIds.indexOf(popupData.station)){

                        popup.remove();

                        map.removeFeatureState({
                            source: 'test-circles',
                            id: hoveredStation
                        });

                        hoveredStation = stationIds.indexOf(popupData.station);

                        if(hoveredStation != stationIds.indexOf(least.station)) {
                                map.setFeatureState({
                                source: 'test-circles',
                                id:hoveredStation
                            },
                            {
                                hover: true
                            });
                        }

                        popupDataToPass = dataMap.get(popupData.station);


                        if(popupData.station == least.station){
                            popup.addClassName('last-record');
                            popup.removeClassName('not-last-record');
                        }
                        else {
                            popup.removeClassName('last-record');
                            popup.addClassName('not-last-record');
                        }
                        popup
                            .setLngLat([popupData.longitude,popupData.latitude])
                            .setHTML(popupCreate(popupDataToPass))
                            .addTo(map)
                    }
                }

                else {
                    map.setFeatureState(
                    {
                        source: 'test-circles',
                        id: hoveredStation
                    },
                    {
                        hover: false
                    }
                    );
                    hoveredStation = null;
                    popup.remove();
                }


            });

        })
	}

    function handleClick(mapLayer) {

        let toHide = layers.filter(d => d != mapLayer);
        toHide = toHide.concat(layersMin.filter(d => d != mapLayer));

        for (let layer in toHide){

            map.setLayoutProperty(
                `acis-${toHide[layer]}`,
                'visibility',
                'none'
            );

            map.setLayoutProperty(
                `acis-${toHide[layer]}-2`,
                'visibility',
                'none'
            );

            map.setLayoutProperty(
                `acis-${toHide[layer]}-3`,
                'visibility',
                'none'
            );

        }

		map.setLayoutProperty(
            `acis-${mapLayer}`,
            'visibility',
            'visible'
        );

        map.setLayoutProperty(
            `acis-${mapLayer}-2`,
            'visibility',
            'visible'
        );

        map.setLayoutProperty(
            `acis-${mapLayer}-3`,
            'visibility',
            'visible'
        );
	}

	onDestroy(() => {
		if (map) map.remove();
	});
</script>

<!-- this special element will be explained in a later section -->
<svelte:head>
	<link
		rel="stylesheet"
		href="https://unpkg.com/mapbox-gl/dist/mapbox-gl.css"
		on:load={load}
	/>
</svelte:head>
<!--
<p>Day-time Max</p>
{#each layers as layer}
    <button on:click={() => handleClick(layer)}>{layer}</button>
{/each}

<p>Night-time Max</p>
{#each layersMin as layer}
    <button on:click={() => handleClick(layer)}>{layer}</button>
{/each} -->


<div class="mapbox-container" bind:this={container}>
	{#if map}
		<slot />
	{/if}
</div>
<div class="mobile-pop-up">
    <p class="mobile-pop-up-content">{@html popupCreate(popupDataToPass)}</p>
</div>

<style>
	.mapbox-container {
		width: 100%;
        height: 100%;
	}

    .mobile-pop-up {
        display: none;
        width: calc(100% - 20px);
        margin: 0 auto;
        box-shadow: 0 0 6px 0 rgba(0,0,0,.35);
        margin-bottom: 1rem;
        margin-top: 1rem;
        padding: .5rem;
        min-height: 100px;
    }

    .mobile-pop-up-content {
        font-family:'Yantramanav';
        font-size: 18px;
        text-align: center;
        line-height: 1.2;
        margin: 0;
        align-self: center;
    }

    @media only screen and (max-width: 500px) {
        .mobile-pop-up {
            display: flex;
        }
    }
</style>
