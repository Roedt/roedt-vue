<template>
    <div id="map-view">
        <h1> {{ header }} </h1>
        <div id="map">
            <l-map ref="mapz" :zoom="zoom" :center="center" :maxZoom="maxZoom">
                <l-tile-layer :url="mapURL" :id="mapID" :attribution="attribution"></l-tile-layer>
                <Vue2LeafletMarkerCluster>
                <LMarker v-for="event in events" v-if="event.latlng !== null" :lat-lng="event.latlng" :key="event.url+event.host">
                    <LPopup :content="event.popup" />
                </LMarker>
                </Vue2LeafletMarkerCluster>
            </l-map>
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import {LMap, LTileLayer, LMarker, LPopup} from 'vue2-leaflet';
import Vue2LeafletMarkerCluster from 'vue2-leaflet-markercluster';

import L from 'leaflet';
import 'leaflet-defaulticon-compatibility';

import APICaller from '../components/APICaller.vue';
import SingleEvent from '../types/SingleEvent.vue';

Vue.component('Vue2LeafletMarkerCluster', Vue2LeafletMarkerCluster);

export default Vue.extend({
    name: 'Map',
    components: {
        LMap, LTileLayer, LMarker, LPopup,
    },
    data: () => ({
     zoom: 4,
     center:  [65.1, 18.0],
     mapID: 'mapbox/streets-v11',
    // tslint:disable-next-line:max-line-length
     attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
    // tslint:disable-next-line:max-line-length
     accessToken: 'pk.eyJ1IjoibWFkc29waGVpbSIsImEiOiJjanZhcnVwMHIxMW15NGVwOHp5aGFldGRuIn0.wHzNVsUQ_ty5tEkrB8vnLQ',
     mapURL: '',
     mapURL2: 'http://{s}.tile.osm.org/{z}/{x}/{y}.png',
     maxZoom: 20,
     events: [] as SingleEvent[],
    }),
    props: ['header'],
    mounted() {
        this.setMapURL();
        const caller = new APICaller();
        this.$nextTick(() => {
            caller.tick((e: SingleEvent[]) => this.createMarkers(e));
            this.locateUser();
        });
    },
    methods: {
        locateUser() {
            navigator.geolocation.getCurrentPosition(this.locateToUser);
        },
        locateToUser(pos: GeolocationPosition) {
            (this.$refs.mapz as LMap).mapObject.setView([pos.coords.latitude, pos.coords.longitude], 11);
        },
        setMapURL() {
            // tslint:disable-next-line:max-line-length
            this.mapURL = 'https://api.mapbox.com/styles/v1/' + this.mapID + '/tiles/{z}/{x}/{y}?access_token=' + this.accessToken;
        },
        createMarkers(events: SingleEvent[]) {
            events.forEach((event: SingleEvent) => this.createMarker(event));
        },
        createMarker(event: SingleEvent) {
            if (event.latlng === undefined) {
                return;
            }
            this.events.push(event);
        },
    },
});

</script>

<style lang="scss" scoped>

@import "~leaflet/dist/leaflet.css";
@import "~leaflet-defaulticon-compatibility/dist/leaflet-defaulticon-compatibility.webpack.css";
@import "~leaflet.markercluster/dist/MarkerCluster.css";
@import "~leaflet.markercluster/dist/MarkerCluster.Default.css";

@media screen and (max-width: 720px) {
    div#map {
        width: 100%;
        height: 100%;
    }
}

div#map{
    height: 38rem;
    border: 1px solid blue;
    margin: 0 auto;
    display: flex;
    justify-content: center;
    min-width: 50vw;
    min-height: 40vw;
}
</style>