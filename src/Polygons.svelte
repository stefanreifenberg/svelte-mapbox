<script>
	import { onMount } from 'svelte';
	import { mapbox } from './mapbox.js';
	import 'mapbox-gl/dist/mapbox-gl.css';

	let container;
	let map;

	onMount(() => {

		const map = new mapbox.Map({
			container: 'map',
			style: {
				'version': 8,
				'sources': {
				'raster-tiles': {
				'type': 'raster',
				'tiles': [
					'https://basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png',
			],
			'tileSize': 256,
			'attribution': '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>'
			
			}
			},
			'layers': [
                {
                'id': 'simple-tiles',
                'type': 'raster',
                'source': 'raster-tiles',
                'minzoom': 0,
                'maxzoom': 22,
                "paint" : {
                        "raster-opacity" : 1
                    }
                }
			]
			},
                center: [ 7.82, 47.995], // starting position
                zoom: 11 // starting zoom
			});
		
		map.on('load', () => {
			map.addSource('bezirke', {
			type: 'geojson',
			data: 'stadtteile.json'
			});
			
			map.addLayer({
				'id': 'bezirke-layer',
				'type': 'fill',
				'source': 'bezirke',
				'layout': {},
                'paint': {
                    'fill-color': '#0080ff', // blue color fill
                    'fill-opacity': 0.2
                }
				});
            map.addLayer({
                'id': 'outline',
                'type': 'line',
                'source': 'bezirke',
                'layout': {},
                'paint': {
                    'line-color': '#000',
                    'line-width': 3
                }
                });
            // Create our popup
            const popup = new mapbox.Popup({
                closeButton: false,
                closeOnClick: false
            });
            
			// Add a mousemove event to the map
            map.on('mousemove', 'bezirke-layer', (e) => {
            	// Change the cursor style as a UI indicator.
                map.getCanvas().style.cursor = 'pointer';
            
            	// Get the mouse coordinates
                const coordinates = e.lngLat.wrap();
				// Get the feature from the layer
                const description = e.features[0].properties.name;
            
            // Populate the popup and set its coordinates
            // based on the feature found.
                popup.setLngLat(coordinates).setHTML('<h2>' + description + '</h2>').addTo(map);
            });

            // Remove everything on mouseleave
            map.on('mouseleave', 'bezirke-layer', () => {
                map.getCanvas().style.cursor = '';
                popup.remove();
            });    

			});
	});
</script>

<div id="map" bind:this={container}></div>

<div class="map-overlay">
	<div class="map-overlay-inner">
        <h1>Districts of Freiburg im Breisgau</h1>
        <p>
            The city of Freiburg im Breisgau is divided into 28 districts. The formerly independent communities of Ebnet im Dreisamtal, Hochdorf im Breisgau, Kappel im Tal, Lehen im Breisgau, Munzingen, Opfingen, Tiengen am Tuniberg and Waltershofen, which were recently incorporated as part of the district reform in Baden-Württemberg, were introduced to a local constitution.
        </p>
		<p>
			<a href="https://geodaten.freiburg.de/geonetwork/srv/ger/catalog.search#/metadata/89141cd9-e688-4c7c-ba85-fe4ea776984d">Data: Freiburg geodata catalog</a>
		</p>
		<p>
			Author: kaldera // Stefan <br>
			<a href="https://github.com/stefanreifenberg/30daymap">Code can be found here</a>
		</p>
	</div>
</div>

<style>
	#map {
		height: 99vh;
	}
	.map-overlay {
		font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
		position: absolute;
		width: 450px;
		height: fit-content;
		top: 0;
		left: 0;
		padding: 10px;
	}
 
	.map-overlay .map-overlay-inner {
		background-color: #fff;
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
		border-radius: 3px;
		padding: 10px;
		margin-bottom: 10px;
        text-align: center;
	}
 
	.map-overlay h1 {
		line-height: 24px;
		display: inline-block;
		margin: 0 0 10px;
	}
</style>