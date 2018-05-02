<template>
  <div id="app">
    <div id="stats">
      <div id="inUse">
        In use: {{ inUse(bicycles).length }}
      </div>
      <div id="available">
        Available: {{ available(bicycles).length }}
      </div>
    </div>
    <l-map :zoom="zoom" :center="center">
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <l-marker :lat-lng="toLatLng(bike)" v-for="bike in inUse(bicycles)" :key="bike.id"></l-marker>
    </l-map>
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker } from "vue2-leaflet";
import firebase from "firebase";

const firebaseApp = firebase.initializeApp({
  databaseURL: "https://pony-bikes-f8cf9.firebaseio.com"
});
const db = firebaseApp.database();

export default {
  name: "App",
  components: { LMap, LTileLayer, LMarker },
  methods: {
    toLatLng(bike) {
      return L.latLng(bike.latitude, bike.longitude);
    },
    inUse(bikes) {
      return bikes.filter(b => b.status === "IN_USE");
    },
    available(bikes) {
      return bikes.filter(b => b.status === "AVAILABLE");
    }
  },
  data() {
    return {
      bicycles: [],
      zoom: 13,
      center: L.latLng(47.471162, -0.551826),
      url:
        "https://api.mapbox.com/v4/mapbox.streets/{z}/{x}/{y}{r}.png?access_token=pk.eyJ1IjoicGllcnJpY2twIiwiYSI6ImNqZ21ldDRhMDFnZGczM3BhM2M4bXB2dmgifQ.sYk2UQOyEy0adoOXP_Rd7g",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    };
  },
  firebase: {
    bicycles: db.ref("/rest/bicycles")
  }
};
</script>

<style>
@import "/../node_modules/leaflet/dist/leaflet.css";

html,
body,
#app {
  height: 100%;
  width: 100%;
  margin: 0;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#stats {
  position: absolute;
  top: 0;
  right: 0;
  z-index: 10000;
}

#stats > div {
  padding: 5px 10px;
  text-align: right;
}

#inUse {
  background-color: yellowgreen;
  color: white;
}

#available {
  background-color: lightskyblue;
  color: white;
}
</style>
