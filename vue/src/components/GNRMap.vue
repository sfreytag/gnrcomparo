<!--

GNRMap
-------

This is a component to wrap on OpenLayers map, which shows the GNR race route.

-->

<template>
    <div id="gnr-map" class="w-100"></div>
</template>

<style lang="scss" scoped>
    #gnr-map {
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
    import ImageStatic from 'ol/source/ImageStatic.js';
    import ImageLayer from 'ol/layer/Image.js';

    // This extent is that of the image of the GNR race route overlaid on the
    // Stamen tiles.
    const extent = [
        -181895.79287416776,
        7351023.954302978,
        -152596.52446960614,
        7360105.62872611
    ];

    // The OpenLayers map.
    let map;

    // Layers
    let gpx_layer;
    let marker_layer;

    // A reference to this component
    let component;

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
        let gnr_src = new VectorSource({
            url: '/gpx/gnr.gpx',
            format: new GPX()
        });

        gnr_src.once("change", function(evt) {
            setCenter(0);
        });
        gpx_layer = new VectorLayer({source: gnr_src,});

        const stamen = new TileLayer({
            source: new Stamen({
                layer: 'toner'
            })
        });
        stamen.setOpacity(0.4);

        let gnr_layer = new ImageLayer({
            source: new ImageStatic({
                url: '/img/gnr-trans.png',
                imageExtent: extent
            })
        });
        gnr_layer.setExtent(extent);

        marker_layer = new VectorLayer({
            source: new VectorSource({})
        });

        map = new Map({
            target: document.getElementById('gnr-map'),
            layers: [gpx_layer, stamen, gnr_layer, marker_layer],
            controls: defaultControls(),
            view: new View({center: [0, 0], zoom: 14})
        });
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
        },
        watch: {
            distance: function(newVal) {
                setCenter(newVal / 21.1)
            }
        }
    }
</script>