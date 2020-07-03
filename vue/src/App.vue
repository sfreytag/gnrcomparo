<!--

GNRComparo
----------

The parent Vue component for this app, that martials the tree of child
components and deals with user interaction.

-->

<template>
    <div class="gnr-cont">
        <div class="d-flex flex-column justify-content-between p-2 gnr-comparo">
            <div class="row1 mb-2">
                <div class="card border-primary h-100">
                    <div class="card-body p-0 h-100">
                        <your-map
                            v-if="mode.current == mode.map"
                            :distance="distance"
                            :gpx-data="gpxData"
                            @gpx-render="onGpxRender">
                        </your-map>

                        <gpx-loader
                            v-if="mode.current == mode.loader"
                            @file-load="onFileLoad">
                        </gpx-loader>
                    </div>
                </div>
            </div>

            <div class="row2 mb-2">
                <div class="card border-primary h-100">
                    <div class="card-body h-100 p-0">
                        <gnr-map :distance="distance"></gnr-map>
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
                    <h4>{{formattedDistance}}km</h4>
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
    import GPXLoader from './GPXLoader.vue';
    import YourMap from './YourMap.vue';
    import GNRMap from './GNRMap.vue';

    export default {
        components: {
            'gpx-loader': GPXLoader,
            'your-map': YourMap,
            'gnr-map': GNRMap
        },
        data: function() {
            return {
                mode: {
                    current: 'loader',
                    loader: 'loader',
                    map: 'map',
                },
                distance: 0.0,
                gpxData: '',
            }
        },
        computed: {
            formattedDistance: function() {
                return this.distance.toFixed(1)
            }
        },
        methods: {
            onFileLoad: function(data) {
                this.gpxData = data;
                this.showYourMap();
            },
            onGpxRender: function(length) {
                this.your_length = length;
            },
            showYourMap: function() {
                this.mode.current = this.mode.map;
            },
            showGpxLoader: function() {
                this.mode.current = this.mode.loader;
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

                if (fwds) {
                    if (d > this.your_length) return;
                    if (d > 21.1) return;
                }
                else {
                    if (d < 0) return;
                }

                this.distance = d;
            },
            refresh: function() {
                this.showGpxLoader();
                this.distance = 0;
            }
        }
    }
</script>
