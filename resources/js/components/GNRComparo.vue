<template>
<div class="gnr-cont">
    <div class="d-flex flex-column justify-content-between p-2 gnr-comparo">
        <div class="row1 mb-2">
            <div class="card border-primary h-100">
                <div class="card-body p-0 h-100">
                    <div id="your-map"
                        class="w-100"
                        v-show="mode.current == mode.map">
                    </div>

                    <div class="p-2"
                        v-show="mode.current == mode.tools">
                        <div class="d-flex justify-content-between align-items-center">
                            <h1 class="text-primary font-weight-light">GNR Comparo!</h1>
                            <a href="https://fortybyforty2019.com/" class="p-2"><img src="/img/40X40.png" width="100" /></a>
                        </div>
                        <p>
                            Compare your run with the Great North Run! See how far through
                            you got. Which of your local landmarks match up to 
                            iconic things on the route? Maybe passing the corner shop is like
                            crossing the Tyne Bridge?

                            Begin with uploading a GPX file from your run.
                        </p>
                        <form>
                            <div class="form-group">
                                <input type="file" ref="gpxFiles"
                                    class="form-control-file btn btn-primary"
                                    @change="handleFile">
                            </div>
                        </form>
                        <p>
                            Enjoying using GNR Comparo? Please donate
                            to our <a href="https://www.justgiving.com/companyteams/fortybyforty">team 40X40 page.</a>
                        </p>
                        <p>
                            Find out how to download a GPX file from Strava <a href="https://support.strava.com/hc/en-us/articles/216918437-Exporting-your-Data-and-Bulk-Export">here</a>. Other fitness apps should have the same functionality!
                        </p>
                        <p class="border-top">
                            <small>GNR Comparo is built on the 2018 race map. The 2019 race might be a bit different.</small>
                        </p>
                    </div>
                </div>
            </div>
        </div>

        <div class="row2 mb-2">
            <div class="card border-primary h-100">
                <div class="card-body h-100 p-0">
                    <div id="gnr-map" class="w-100"></div>
                </div>
            </div>
        </div>

        <div id="controls"
            class="d-flex justify-content-center align-items-stretch"
            v-if="mode.current == mode.map">
            <div class="btn-group">
                <button class="btn btn-primary" @click="back">
                    <h4 class="font-weight-light mb-0"><i class="fas fa-backward"></i></h4>
                </button>
                <button class="btn btn-primary" @click="fwds">
                    <h4 class="font-weight-light mb-0"><i class="fas fa-forward"></i></h4>
                </button>
            </div>
            <div class="bg-light d-inline-block distance p-2 border border-primary">
                <h4>{{distance}}km</h4>
            </div>
            <button class="btn btn-primary"
                @click="refresh"
                title="Start again and load a different run...">
                <h4 class="font-weight-light mb-0"><i class="fas fa-redo-alt"></i></h4>
            </button>
        </div>
    </div>
</div>
</template>

<style lang="scss" scoped>

.gnr-cont {
    position: absolute;
    top: 0;
    bottom: 20px;
    left: 0;
    right: 0;
}

.gnr-comparo {

    max-width: 800px;
    margin: auto;
    height: 100%;

    .row1 {
        height: 48%;
        .card {
            overflow: scroll;
        }
    }
    .row2 {
        height: 48%;
    }
    #gnr-map, #your-map {
        height: 100%;
    }
    #controls {
        position: absolute;
        top: calc(50% - 30px);
        left: 0;
        right: 0;
        height: 50px;
    }
}

</style>

<script>
    import {Map, View} from 'ol';
    import {Tile as TileLayer, Vector as VectorLayer} from 'ol/layer.js';
    import XYZ from 'ol/source/XYZ.js';
    import GPX from 'ol/format/GPX.js';
    import VectorSource from 'ol/source/Vector.js';
    import ImageLayer from 'ol/layer/Image.js';
    import ImageStatic from 'ol/source/ImageStatic.js';
    import Stamen from 'ol/source/Stamen.js';
    import {defaults as defaultControls} from 'ol/control.js';
    import {Icon, Stroke, Style} from 'ol/style.js';
    import Feature from 'ol/Feature.js';
    import Point from 'ol/geom/Point.js';
    import {getLength} from 'ol/sphere.js';

    const extent = [
        -181895.79287416776,
        7351023.954302978,
        -152596.52446960614,
        7360105.62872611
    ];

    const styles = {
        'marker': new Style({
            image: new Icon({
                anchor: [0.5, 1],
                src: '/img/marker.png',
                size: [64, 64]
            })
        })
    };

    // Layers
    let your_gpx;
    let your_map;
    let your_marker;

    let gnr_map;
    let gnr_gpx;
    let gnr_marker;

    function buildYourMap() {
        your_gpx = new VectorLayer({});

        your_marker = new VectorLayer({
            source: new VectorSource({})
        });

        const stamen = new TileLayer({
            source: new Stamen({
                layer: 'toner'
            })
        });
        stamen.setOpacity(0.4);

        your_map = new Map({
            target: document.getElementById('your-map'),
            layers: [stamen, your_gpx, your_marker],
            controls: defaultControls(),
            view: new View({center: [
                -179000,
                7358000
            ], zoom: 14})
        });
        refreshMaps();
    }

    function buildGnrMap() {
        let gnr_src = new VectorSource({url: '/gpx/gnr.gpx', format: new GPX() });
        gnr_src.once("change", function(evt) {
            setGnrCenter(0);
        });
        gnr_gpx = new VectorLayer({source: gnr_src,});

        const stamen = new TileLayer({
            source: new Stamen({
                layer: 'toner'
            })
        });
        stamen.setOpacity(0.4);

        let gnr = new ImageLayer({
            source: new ImageStatic({
                url: '/img/gnr-trans.png',
                imageExtent: extent
            })
        });
        gnr.setExtent(extent);

        gnr_marker = new VectorLayer({
            source: new VectorSource({})
        });

        gnr_map = new Map({
            target: document.getElementById('gnr-map'),
            layers: [gnr_gpx, stamen, gnr, gnr_marker],
            controls: defaultControls(),
            view: new View({center: [0, 0], zoom: 14})
        });

        refreshMaps();
    }

    function refreshMaps() {
        //return;

        const h = window.outerHeight;
        window.outerHeight = h-1;
        window.setTimeout(function() {
            window.outerHeight = h;
            if (gnr_map) {
                gnr_map.updateSize();
                gnr_map.renderSync();
            }
            if (your_map) {
                your_map.updateSize();
                your_map.renderSync();
            }
        }, 100);
    }

    function setGnrCenter(fraction) {
        let geom = gnr_gpx.getSource().getFeatures()[0].getGeometry();
        if (geom.getType() == "MultiLineString")
            geom = geom.getLineStrings()[0];
        const coord = geom.getCoordinateAt(fraction);
        gnr_map.getView().setCenter(coord);
        drawFeature(gnr_marker, coord);
    }

    function getYourLine() {
        let geom = your_gpx.getSource().getFeatures()[0].getGeometry();
        if (geom.getType() == "MultiLineString")
            geom = geom.getLineStrings()[0];
        return geom;
    }

    function setYourCenter(fraction) {
        const geom = getYourLine();
        const coord = geom.getCoordinateAt(fraction);
        your_map.getView().setCenter(coord);
        drawFeature(your_marker, coord);
    }

    function drawFeature(layer, coord) {
        let feature;
        if  (layer.getSource().getFeatures().length > 0) {
            feature = layer.getSource().getFeatures()[0];
        }
        else {
            feature = new Feature({type: 'marker'});
            feature.setStyle(styles.marker);
            layer.getSource().addFeature(feature);
        }
        feature.setGeometry(new Point(coord));
        //refreshMaps();
    }

    export default {
        data: function() {
            return {
                mode: {
                    current: 'map',
                    tools: 'tools',
                    map: 'map',
                },
                distance: '0.0'
            }
        },
        methods: {
            handleFile: function() {
                const file = this.$refs.gpxFiles.files[0];
                if (!file) return;
                let reader = new FileReader();
                reader.addEventListener("load", function () {
                    this.createFromGPX(reader.result);
                }.bind(this), false)
                reader.readAsDataURL(file);
            },
            createFromGPX: function(data) {
                this.showYourMap();
                const src = new VectorSource({
                    format: new GPX(),
                    url: data,
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
                        i++
                    }

                    geom.setCoordinates(trimmed);
                    feature.setGeometry(geom);

                    setYourCenter(0);
                    refreshMaps();
                });

                your_gpx.setSource(src);
            },
            showYourMap: function() {
                this.mode.current = this.mode.map;
                
            },
            showTools: function() {
                this.mode.current = this.mode.tools;
            },
            fwds: function() {
                this.go(true);
            },
            back: function() {
                this.go(false);
            },
            go: function(fwds) {
                const delta = fwds ? 0.1 : -0.1;
                let d = parseFloat(this.distance);
                d = d + delta;
                d = Math.round(d * 10) / 10;

                const line = getYourLine();
                const your_length = Math.round(getLength(line) / 1000);
                if (fwds) {
                    if (d > your_length) return;
                    if (d > 21.1) return;
                }
                else {
                    if (d < 0) return;
                }

                setGnrCenter(d / 21.1);
                setYourCenter(d / your_length);

                this.distance = d.toFixed(1);
            },
            refresh: function() {
                this.$refs.gpxFiles.value = '';
                this.showTools();
                this.distance = "0.0";
                setGnrCenter(0);
            }
        },
        mounted() {
            buildGnrMap();
            buildYourMap();
            refreshMaps();
            window.setTimeout(function() {
                this.showTools();
            }.bind(this), 100);
        }
    }
</script>
