<template>
  <div class="" style="height: 30vh">
    <h1>Search a State! - Mingxin Yu</h1>
    <input v-model="key" v-on:input="mounted()" placeholder="State Name" />
    <div v-on:click="selectState(state)" v-for="state in states" :key="state.name">
      {{ state.name }}
    </div>
  </div>
  <div id="map" style="width: 100%; height: 70vh" />
</template>

<script>
/* eslint-disable no-undef */
import axios from "axios";
import { onMounted } from 'vue'
import { Loader } from '@googlemaps/js-api-loader'

const GOOGLE_MAPS_API_KEY = 'AIzaSyBiW4KbgNz-DQMJMpIbOOFdQid1A5bRd40'

export default {
    name: "SearchState",
    setup() {
        const loader = new Loader({ apiKey: GOOGLE_MAPS_API_KEY })
        onMounted(async () => {
        await loader.load()
        })
    },
    data() {
        return {
            key: '',
            states: [],
            map: null,
        };
    },
    methods: {
        initMap() {
            this.map = new google.maps.Map(document.getElementById("map"), {
            center: { lat: 37.09024, lng: -95.712891 },
            zoom: 6,
            });
        },
        async mounted() {
            try {
                if (this.key === '') {
                    this.states = [];
                } else {
                    var query = ` { getAllStates(key: "` + this.key +  `") { name } }`;
                        var request = {
                            method: "POST",
                            url: "http://localhost:6969/graphql",
                            data: { query: query}
                        }
                    var result = await axios(request);
                    this.states = result.data.data.getAllStates;
                }
            } catch (error) {
                console.error(error);
            }
        },
        getLatLng() {
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?address="
                + this.key
                + "&key="
                +  GOOGLE_MAPS_API_KEY)
            .then(response => {
                if (response.data.error_message) {
                    console.log(response.data.error_message);
                } else {
                    this.moveToLocation(
                    response.data.results[0].geometry.location.lat,
                    response.data.results[0].geometry.location.lng)
                }})
            .catch(error => {
                console.log(error.message);
            })
        },
        moveToLocation(Lat, Lng){
            var center = new google.maps.LatLng(Lat, Lng);
            this.map.panTo(center);
        },
        selectState(state){
            this.key = state.name;
            this.states = [];
            this.getLatLng();
        },
    }
}
</script>

<style></style>