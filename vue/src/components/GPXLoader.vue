<!--

GPXLoader
---------

Provides some intro text and lets the user load a GPX file to begin the comparo.

-->

<template>
    <div class="p-2">
        <div class="d-flex justify-content-between align-items-center">
            <h1 class="text-primary font-weight-light">GNR Comparo!</h1>
            <a href="https://www.justgiving.com/teams/fortybyforty" class="p-2"><img src="/img/40X40.png" width="100" /></a>
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
            <small>GNR Comparo is built on the 2018 race map. But the course should be pretty much the same nowadays!</small>
        </p>
    </div>
</template>

<script>
    export default {
        methods: {
            handleFile: function() {
                const file = this.$refs.gpxFiles.files[0];
                if (!file) return;
                let reader = new FileReader();
                reader.addEventListener("load", function () {
                    this.$emit('file-load', reader.result);
                    this.$refs.gpxFiles.value = '';
                }.bind(this), false)
                reader.readAsDataURL(file);
            },
        },
    }
</script>