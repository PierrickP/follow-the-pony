<template>
  <div id="app">
    <div id="stats">
      <div id="inUse">
        In use: {{ inUse(bicycles).length }}
      </div>
      <div id="available">
        Available: {{ available(bicycles).length }}
      </div>
      <div id="cities">
        <input type="radio" id="Angers" value="Angers" v-model="city">
        <label for="Angers">Angers</label>
        <br>
        <input type="radio" id="Oxford" value="Oxford" v-model="city">
        <label for="Oxford">Oxford</label>
      </div>
    </div>
    <l-map ref="map"  :zoom="zoom" :center="center">
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>

      <l-marker :lat-lng="toLatLng(bike)" v-for="bike in inUse(bicycles)" :key="bike.physicalId" :icon="getIcon('in-use', bike.adopted)">
        <l-popup>
          <popup-pony :data="bike"/>
        </l-popup>
      </l-marker>

      <l-marker :lat-lng="toLatLng(bike)" v-for="bike in available(bicycles)" :key="bike.physicalId" :icon="getIcon('available', bike.adopted)">
        <l-popup>
          <popup-pony :data="bike"/>
        </l-popup>
      </l-marker>
    </l-map>
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker, LPopup } from "vue2-leaflet";
import firebase from "firebase";
import PopupPony from "../src/components/PopupPony.vue";

const firebaseApp = firebase.initializeApp({
  databaseURL: "https://pony-bikes-f8cf9.firebaseio.com"
});
const db = firebaseApp.database();
const cities = {
  Angers: [47.471162, -0.551826],
  Oxford: [51.752022, -1.257726]
};

export default {
  name: "App",
  components: { LMap, LTileLayer, LMarker, LPopup, PopupPony },
  methods: {
    toLatLng(bike) {
      return L.latLng(bike.latitude, bike.longitude);
    },
    inUse(bikes) {
      // console.log(bikes);
      return bikes.filter(b => b.status === "IN_USE" && b.region === this.city);
    },
    available(bikes) {
      return bikes.filter(
        b => b.status === "AVAILABLE" && b.region === this.city
      );
    },
    getContent(b) {
      return JSON.stringify(b);
    },
    getIcon(type, adopted) {
      return L.icon({
        prefix: "",
        iconUrl: `/static/${type}${adopted ? "-adopted" : ""}.png`,
        iconSize: type === "available" ? [10, 10] : [24, 36]
      });
    }
  },
  data() {
    return {
      map: null,
      city: "Angers",
      bicycles: [],
      zoom: 13,
      center: JSON.parse(JSON.stringify(cities["Angers"])),
      url:
        "https://api.mapbox.com/v4/mapbox.streets/{z}/{x}/{y}{r}.png?access_token=pk.eyJ1IjoicGllcnJpY2twIiwiYSI6ImNqZ21ldDRhMDFnZGczM3BhM2M4bXB2dmgifQ.sYk2UQOyEy0adoOXP_Rd7g",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    };
  },
  firebase: {
    bicycles: db.ref("/rest/bicycles")
  },
  mounted() {
    this.$nextTick(() => {
      this.map = this.$refs.map.mapObject;
    });
  },
  watch: {
    city: function(v) {
      this.map.panTo(JSON.parse(JSON.stringify(cities[v])));
    }
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

#cities {
  background-color: #fff;
  text-align: left !important;
}
</style>
