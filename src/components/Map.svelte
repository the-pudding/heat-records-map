<script>
	import { onDestroy } from 'svelte';
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();

    // import mapboxgl from 'mapbox-gl'; // or "const mapboxgl = require('mapbox-gl');"
    import { onMount } from "svelte";
    import { scaleLinear, selectAll, timeFormat, geoAlbersUsa, geoMercator } from "d3";
    import viewport from "$stores/viewport.js";

    export let data;
    export let least;
    export let timeframe;

    let loaded;
    let sourceLoaded;
    let flyTod;

    let timeoutTime = 3000;
    let speed = .5;


    let variables = {
        "daily":"days_since_daily",
        "all-time":"all_time_days"
    }

    let variablesDate = {
        "daily":"days_since_daily_date",
        "all-time":"all_time_date"
    }

    let variablesVal = {
        "daily":"days_since_val",
        "all-time":"all_time_val"
    }

    let variablesText = {
        "daily":"daily_text",
        "all-time":"all_time_text"
    }

    let variablesOpacity = {
        "daily":[7,30],
        "all-time":[30,365]
    }

    let variablesRadius = {
        "daily":[0,50],
        "all-time":[0,365]
    }

    let layersLoaded = {
        "daily":false,
        "all-time":false,
    }


    mapboxgl.accessToken = 'pk.eyJ1IjoiZG9jazQyNDIiLCJhIjoiY2xrZ3llNmFkMDRpYjNkcDRuNDQ3MDBmayJ9.r7nPJo7xLNkUAYWp7QO9_g';
	// set the context here...

		export let lat;
		export let lon;
		export let zoom;

    const formatTime = timeFormat("%B %d, %Y");
    const formatTimeNoYear = timeFormat("%b. %d");
    const formatTimeYear = timeFormat("%b. %d ’%y");

    let popupDataToPass = least;
    let popupString = "hi"

    $: w = $viewport.width;
    $: h = $viewport.height;

    let layers = [
        "daily","daily-days","monthly","alltime"
    ]

    const getCountFormatDays = (value) => {
        if(value < 2){
            return `${value} Day`
        }
        else{
            return `${value} Days`
        }
    }

    const getCountFormatAllTime = (value) => {
        if(value < 2){
            return `${value} Day`
        }
        else if(value < 366){
            return `${value} Days`
        }
        else{
            let years = Math.floor(value/365);

            if(years == 1){
                return `${years} Yr.`
            }
            else {
                return `${years} Yrs.`
            }

        }
    }


    const getCountFormat = (value) => {
        if(value < 2){
            return `${value} Day Ago`
        }
        else if(value < 365){
            return `${value} Days Ago`
        }
        else {
            return `${value} Yrs. Ago`;
        }

    }

    let layersMin = [
        "daily-night","daily-days-night","monthly-night","alltime-night"
    ]

    function formatDate(dataToMake){
        let dateNoSuffix = new Date(dataToMake[variablesDate[timeframe]].slice(0,4), +dataToMake[variablesDate[timeframe]].slice(5,7) -1, dataToMake[variablesDate[timeframe]].slice(8,10));
        return formatTimeNoYear(dateNoSuffix);
    }

    function formatDateGeoJson(dataToMake){
        let dateNoSuffix = new Date(dataToMake.slice(0,4), dataToMake.slice(5,7) -1, dataToMake.slice(8,10));
        return formatTimeYear(dateNoSuffix);
    }

    let mounted = false;

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


    let center = [lon,lat];

    let stationIds = data.map(function(d){
        return d.station;
    })

    let popup = null;
    let padding = {top:50};
    let navPosition = 'top-right';
    let scrollZoom = true;
    let initialZoom = zoom;
    let geojson;

	function load() {

        geojson = {
                "type":"FeatureCollection",
                "features":null
        };

        geojson.features = data.map(d => {            
            return {
                "type":"Feature",
                "properties":{
                    "station":d.station,
                    "all_time_days":+d.all_time_days,
                    "daily_text":getCountFormatDays(+d.days_since_daily),
                    "all_time_text":getCountFormatAllTime(+d.all_time_days),
                    "days_since_daily":+d.days_since_daily,
                    "name":d.name.replace(" Area",""),
                    "days_since_val":d.days_since_val,
                    "all_time_val":d.all_time_val,
                    "days_since_daily_date":formatDateGeoJson(d.days_since_daily_date),
                    "all_time_date":formatDateGeoJson(d.all_time_date)
                },
                "id":stationIds.indexOf(d.station),
                "geometry":{
                    "type":"Point",
                    "coordinates":[d.true_longitude,d.true_latitude]
                }
            }
        })

        if(!map){

            let style = "cl0bhzae7000y14utgxgy71f1";
            style = "clkhcazsh00jj01qwcg723m24";

            map = new mapboxgl.Map({
                container,
                style: `mapbox://styles/dock4242/${style}`, //'mapbox://styles/dock4242/cl0banu3w000i14l8w1woe65r',
                // maxBounds: maxBounds,
                center: [80, 36],//center,//[-95.973515, 38.382024],//,
                // pitch:17,
                zoom: initialZoom,
                scrollZoom: scrollZoom
		    });
        }
        else {
            map.remove();
            map = new mapboxgl.Map({
                container,
                style: `mapbox://styles/dock4242/${style}`, //'mapbox://styles/dock4242/cl0banu3w000i14l8w1woe65r',
                maxBounds: maxBounds,
                center: center,//[-95.973515, 38.382024],//,
                zoom: initialZoom,
                scrollZoom: scrollZoom,
                pitch: 80
		    });
        }

        let popupAdded = false;
        let hoveredStation = null;

        map.once('idle', function(){
            loaded = true;
            map.addControl(new mapboxgl.NavigationControl());
        })
	}

    const loadLayers = () => {

        let sizeControlLength = selectAll(".mapboxgl-ctrl-zoom-in").size();
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

        map.setLayoutProperty("acis-agg-2-986vec-dots","visibility","none");
        map.setLayoutProperty("acis-agg-2-986vec","visibility","none");

        if(layersLoaded["daily"]){
        }
        
        if(!sourceLoaded){
            sourceLoaded = true;
            map.addSource('acis', {
                'type': 'geojson',
                'data': geojson
            });
        }

        if(!layersLoaded[timeframe]){
            map.addLayer({
                'id': `circle-${timeframe}`,
                'type': 'circle',
                'source': 'acis',
                'layout': {},
                'paint': {
                    'circle-stroke-color': '#000',
                    'circle-radius': [
                        "interpolate",["linear"],["zoom"],
                        3,
                        ["match",[ "get", "station" ], [least.station], 10,
                            ["interpolate",["linear"],
                            ["get",variables[timeframe]],
                                variablesRadius[timeframe][0],10,
                                variablesRadius[timeframe][1],2
                            ] ],
                        6,["interpolate",["linear"],["get",variables[timeframe]],0,10,100,10]
                    ],
                    'circle-stroke-width': ["match",[ "get", "station" ], [least.station], 1,0],
                    'circle-color': [
                        "interpolate", ["linear"], ["get", variables[timeframe]],
                        0, "hsl(0, 95%, 57%)", 100, "hsl(0, 0%, 85%)" 
                    ],
                    'circle-stroke-opacity':1
                }
            },"settlement-minor-label"); 

            map.addLayer({
                'id': `label-${timeframe}`,
                'type': 'symbol',
                'source': 'acis',
                'layout': {
                    'text-offset':[
                        "step",
                        ["zoom"],
                        ["literal", [0, 0]],
                        2,
                        ["literal", [0, 0]],
                        6,
                        ["literal", [0, 0.55]]
                    ],
                    'text-field':
                        ["step",
                            ["zoom"],
                            ["match",[
                                    "get",
                                    "station"
                                ],
                                [least.station], //this removes the text for the station we don't want
                                "",
                                [
                                "concat",
                                [
                                    "to-string",
                                    ["get",
                                    timeframe == "daily" ? variables[timeframe] : variablesText[timeframe]
                                    ]
                                ],
                                ""
                                ]
                            ],
                            6,
                            [
                                'format',
                                ["to-string",
                                    ["get",
                                    variablesText[timeframe],
                                    ]
                                ],
                                { 'font-scale': 1 },
                                '\n',
                                {},
                                [ "concat", [ "to-string", ["get", "name"] ], " ", ["get", variablesVal[timeframe]],"°F" ],
                                { 'font-scale': .8 },
                                '\n',
                                {},
                                [ "to-string", ["get", variablesDate[timeframe]] ],
                                { 'font-scale': .7 }
                            ]
                        ]
                    ,
                    "text-size":16,
                    'text-allow-overlap':[
                        "step",
                        ["zoom"],
                        false,
                        5,
                        true
                    ],
                    "text-font":["Overpass Mono SemiBold","Arial Unicode MS Regular"],
                    'text-ignore-placement':[
                        "step",
                        ["zoom"],
                        true,
                        5,
                        false
                    ]
                },
                'paint':{
                    "text-halo-color":"hsl(0, 0%, 100%)",
                    "text-halo-width":1.5,
                    "text-opacity":[
                        "interpolate",
                        ["linear"],["zoom"],
                        4,
                        ["interpolate",["linear"],["get",
                            variables[timeframe]],
                            variablesOpacity[timeframe][0],
                            1,
                            variablesOpacity[timeframe][1],
                            0
                        ],
                        6,
                        ["interpolate",["linear"],["get",
                            variables[timeframe]],14,1,100,1
                        ]
                    ]
                }
            });
        
            map.addLayer({
                'id': `label-top-${timeframe}`,
                'type': 'symbol',
                'source': 'acis',
                'maxzoom':6,
                'filter':[
                    "all",[
                        "match",
                        ["get","station"],
                        [least.station],
                        true,
                        false
                    ]
                ],
                'layout': {
                    'text-line-height':1.2,
                    'text-anchor':"top",
                    'text-padding':20,
                    'text-offset':[0,-.5],
                    'text-field': [
                        'format',
                        `${getCountFormat(least[variables[timeframe]])}`,
                        { 'font-scale': 1},
                        '\n',
                        {},
                        `${least[variablesVal[timeframe]]}°F in ${least.name.replace(" Area","")}`,
                        { 'font-scale': .75 },
                        '\n',
                        {},
                        `Hottest ${timeframe == "daily" ? formatDate(least) : 'Temperature'} Ever`,
                        { 'font-scale': .75 }
                    ],
                    // "text-size":18,
                    "text-font":["Overpass Mono Bold","Arial Unicode MS Regular"],
                },
                'paint':{
                    "text-halo-color":"hsl(0, 0%, 100%)",
                    "text-halo-width":2,
                    "text-opacity":["interpolate",["linear"],["zoom"],4,["interpolate",["linear"],["get",variables[timeframe]],14,1,50,0],6,["interpolate",["linear"],["get",variables[timeframe]],14,1,100,1]]
                }
            });
        }

        let toHide;
        if(timeframe == "daily"){
            toHide = "all-time"
        }
        else {
            toHide = "daily"
        }

        if(layersLoaded[toHide]){
            map.setLayoutProperty(`circle-${toHide}`,"visibility","none");
            map.setLayoutProperty(`label-${toHide}`,"visibility","none");
            map.setLayoutProperty(`label-top-${toHide}`,"visibility","none");
        }
        
        if(layersLoaded[timeframe]){
            map.setLayoutProperty(`circle-${timeframe}`,"visibility","visible");
            map.setLayoutProperty(`label-${timeframe}`,"visibility","visible");
            map.setLayoutProperty(`label-top-${timeframe}`,"visibility","visible");
        }

        map.once('idle',function(){

            layersLoaded[timeframe] = true;

            if(!flyTod){
                setTimeout(async () => {

                    dispatch('loaded', {});

                    map.flyTo({
                        center: [least.true_longitude, least.true_latitude],
                        zoom: 4,
                        pitch:0,
                        speed:speed,
                        essential: true // This animation is considered essential with
                        //respect to prefers-reduced-motion
                    });

                    flyTod = true;
                }, timeoutTime) /* <--- If this is enough greater than transition, it doesn't happen... */
            } else {
                
                if(map.getZoom() < 5){
                    map.flyTo({
                        center: [least.true_longitude, least.true_latitude],
                        zoom: 4,
                        pitch:0,
                        speed:speed,
                        essential: true // This animation is considered essential with
                        //respect to prefers-reduced-motion
                    });
                }

                
            }


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

    $: if(mounted == true) load();
    $: timeframe, loaded ? loadLayers() : '';
</script>

<!-- this special element will be explained in a later section -->
<!-- <svelte:head>
	<link
		rel="stylesheet"
		href="https://unpkg.com/mapbox-gl/dist/mapbox-gl.css"
		on:load={load}
	/>
</svelte:head> -->
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
