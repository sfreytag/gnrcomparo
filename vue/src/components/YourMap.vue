<!--

YourMap
-------

This is a component to wrap on OpenLayers map, which shows your route uploaded
from a GPX file.

-->

<template>
    <div id="your-map"
        class="w-100">
    </div>
</template>

<style lang="scss" scoped>
    #your-map {
        height: 100%;
    }
</style>

<script>
    import {Icon, Stroke, Style} from 'ol/style.js';
    import {defaults as defaultControls} from 'ol/control.js';
    import {Tile as TileLayer, Vector as VectorLayer} from 'ol/layer.js';
    import VectorSource from 'ol/source/Vector.js';
    import Stamen from 'ol/source/Stamen.js';
    import {Map, View} from 'ol';
    import GPX from 'ol/format/GPX.js';
    import Feature from 'ol/Feature.js';
    import Point from 'ol/geom/Point.js';
    import {getLength} from 'ol/sphere.js';

    // The OpenLayers map.
    let map;

    // Layers
    let gpx_layer;
    let marker_layer;

    // A reference to this component
    let component;

    // The length of the GPX line.
    let length = 0;

    // Styles for OpenLayers elements used in the map rendered by this
    // component.
    const styles = {
        'marker': new Style({
            image: new Icon({
                anchor: [0.5, 1],
                src: '/img/marker.png',
                size: [64, 64]
            })
        })
    };

    // buildMap is called when the component is mounted and it builds the
    // OpenLayers maps and layers needed to display your GPX data.
    function buildMap() {
        gpx_layer = new VectorLayer({});

        marker_layer = new VectorLayer({
            source: new VectorSource({})
        });

        const stamen = new TileLayer({
            source: new Stamen({
                layer: 'toner'
            })
        });
        stamen.setOpacity(0.4);

        map = new Map({
            target: document.getElementById('your-map'),
            layers: [stamen, gpx_layer, marker_layer],
            controls: defaultControls(),
            view: new View({center: [-179000, 7358000], zoom: 14})
        });
    }

    // loadGpxData watches the gpx-data prop, and when it changes, takes the
    // supplied data URL and creates a GPX vector source from it.
    function loadGpxData() {
        const src = new VectorSource({
            format: new GPX(),
            url: component.gpxData
        });

        src.once("change", function(evt) {
            let feature = src.getFeatures()[0];
            let geom = feature.getGeometry();
            if (geom.getType() == "MultiLineString")
                geom = geom.getLineStrings()[0];
            const coords = geom.getCoordinates();

            const retain = 5;
            let coord = 0, i = 0, trimmed = [];
            while (coord = coords[i]) {
                if (i % retain == 0) trimmed.push(coord);
                i++;
            }

            geom.setCoordinates(trimmed);
            feature.setGeometry(geom);

            length = getLength(geom) / 1000;
            component.$emit('gpx-render', length);

             setCenter(0);
        });

        gpx_layer.setSource(src);
    }

    // setCenter sets the map center to a point along the GPX line, based on the
    // value of the fraction, which is between 0 and 1.
    function setCenter(fraction) {
        if (fraction < 0 || fraction > 1) return;
        const geom = getLineString();
        const coord = geom.getCoordinateAt(fraction);
        map.getView().setCenter(coord);
        drawFeature(coord);
    }

    // drawFeature positions a marker icon at a point along the GPX line
    // identified by the OL coord.
    function drawFeature(coord) {
        let feature;
        if  (marker_layer.getSource().getFeatures().length > 0) {
            feature = marker_layer.getSource().getFeatures()[0];
        }
        else {
            feature = new Feature({type: 'marker'});
            feature.setStyle(styles.marker);
            marker_layer.getSource().addFeature(feature);
        }
        feature.setGeometry(new Point(coord));
    }

    // getLineStringGeom
    // This gets the LineString geometery created by the GPX VectorSource. It 
    // allows for the fact that a GPX may be bundled as a multi line string, in
    // which case, it extracts the first.
    function getLineString() {
        let geom = gpx_layer.getSource().getFeatures()[0].getGeometry();
        if (geom.getType() == "MultiLineString")
            geom = geom.getLineStrings()[0];
        return geom;
    }

    // Export the component.
    export default {
        props: {
            gpxData: String,
            distance: Number
        },
        mounted: function() {
            component = this;
            buildMap();
            loadGpxData();
        },
        watch: {
            distance: function(newVal) {
                setCenter(newVal / length);
            }
        }
    }
</script>