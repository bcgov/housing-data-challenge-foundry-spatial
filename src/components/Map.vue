<template>
<div class="map-wrap">
    <div id="map"></div>
</div>
</template>

<script>
// import libraries
import mapboxgl from 'mapbox-gl/dist/mapbox-gl';

// config
import config from '../app.config';
// vuex store
import store from '../store';

import mapColors from '../mapColors';

export default {
    name: 'map-viz',
    data() {
        return {
            map: {},
            layers: config.map.dataLayers,
        };
    },
    mounted() {
        mapboxgl.accessToken = config.mapboxgl.accessToken;
        this.map = new mapboxgl.Map({
            container: 'map',
            style: config.mapboxgl.baseStyle,
            center: config.map.center,
            zoom: config.map.zoom,
        });

        this.map.on('style.load', () => {
            // apply default colors
//            this.updateColors(this.colorField);
            // centralize the map object in the store
            store.commit('setMap', this.map);
            // initialize the data for view information.
        });
        this.map.on('load', () => {
            this.setMapPropertiesMinMax();
        });
        // track map view in the store
        this.map.on('moveend', () => {
            this.setMapPropertiesMinMax();
            const zoom = this.map.getZoom();
            const bounds = this.map.getBounds();
            const bearing = this.map.getBearing();
            const pitch = this.map.getPitch();
            store.commit('changeMapView', {
                zoom,
                bounds,
                bearing,
                pitch,
            });
        });
    },
    computed: {
        colorField() {
            return store.state.mapColorField;
        },
    },
    methods: {
        updateColors(val) {
            const paintProperty = mapColors[val].paintProperty();
            this.layers.forEach((layer) => {
                this.map.setPaintProperty(layer, 'fill-color', paintProperty);
            });
        },
        setMapPropertiesMinMax() {
            // create a list of properties and their min/max values.
            const minmax = {}; // for annoying eslint rules.
            const valueBuckets = {};
            const stops = {};

            // assume the colorField is the only one we want to figure out right now.

            const key = this.colorField;

            minmax[key] = { min: undefined, max: undefined };
            valueBuckets[key] = [];
            const paintProperty = mapColors[key].paintProperty();
            stops[key] = paintProperty.stops;

            // I don't think this actually gives us much of a speed boost
            const tr = this.map.project(this.map.getBounds().getNorthEast());
            const bl = this.map.project(this.map.getBounds().getSouthWest());

            this.map.queryRenderedFeatures(
                [tr, bl],
                { layers: config.map.dataLayers }).forEach((feature) => {
                    valueBuckets[key].push(feature.properties[key]);
                    if (minmax[key].min === undefined
                        || minmax[key].min > feature.properties[key]) {
                        minmax[key].min = feature.properties[key];
                    }
                    if (minmax[key].max === undefined
                        || minmax[key].max < feature.properties[key]) {
                        minmax[key].max = feature.properties[key];
                    }
                });
            const count = stops[key].length;
            const incr = (minmax[key].max - minmax[key].min) / count;
            for (let i = 0; i < count; i += 1) {
                stops[key][i] = [Math.round(minmax[key].min + (incr * i)), stops[key][i][1]];
            }
            this.layers.forEach((layer) => {
                this.map.setPaintProperty(layer, 'fill-color', { property: this.colorField, stops: stops[this.colorField] });
            });
            store.commit('setCurrentViewValues', { extrema: minmax, all: valueBuckets });
        },
    },
    watch: {
        colorField(val) {
            this.updateColors(val);
        },
    },
};
</script>

<style lang="scss">
// import mapboxgl css
@import '~mapbox-gl/dist/mapbox-gl.css';

#map {
    height: 55vh;
}
</style>