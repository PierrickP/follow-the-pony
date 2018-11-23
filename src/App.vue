<template>
  <div id="app">
    <div id="controls">
      <div class="stats">
        <span>Bikes</span>
        <div id="inUse">
          In use: {{ bicyclesInUse }}
        </div>
        <div id="available">
          Available: {{ bicyclesAvailable }}
        </div>
        <div id="available">
          Adopted: {{ bicyclesAdopted }} ({{ Math.round(bicyclesAdopted / bicyclesAvailable * 100) }}%)
        </div>
      </div>
      <div class="stats">
        <span>Scooters</span>
        <div id="inUse">
          In use: {{ scootersInUse }}
        </div>
        <div id="available">
          Available: {{ scootersAvailable }}
        </div>
        <div id="available">
          Adopted: {{ scootersAdopted }} ({{ Math.round(scootersAdopted / scootersAvailable * 100)  }}%)
        </div>
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

      <l-marker :lat-lng="toLatLng(vehicle)" v-for="vehicle in inUse(bicycles)" :key="vehicle.physicalId" :icon="getIcon('bike', 'in-use', vehicle.adopted)">
        <l-popup>
          <popup-pony :data="vehicle"/>
        </l-popup>
      </l-marker>

      <l-marker :lat-lng="toLatLng(vehicle)" v-for="vehicle in available(bicycles)" :key="vehicle.physicalId" :icon="getIcon('bike', 'available', vehicle.adopted)">
        <l-popup>
          <popup-pony :data="vehicle"/>
        </l-popup>
      </l-marker>

      <l-marker :lat-lng="toLatLng(vehicle)" v-for="vehicle in inUse(scooters)" :key="vehicle.vehicleCode" :icon="getIcon('scooter', 'in-use', vehicle.adopted)">
        <l-popup>
          <popup-pony :data="vehicle"/>
        </l-popup>
      </l-marker>

      <l-marker :lat-lng="toLatLng(vehicle)" v-for="vehicle in available(scooters)" :key="vehicle.physicalId" :icon="getIcon('scooter', 'available', vehicle.adopted)">
        <l-popup>
          <popup-pony :data="vehicle"/>
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
    toLatLng(vehicle) {
      return vehicle.position
        ? L.latLng(vehicle.position.latitude, vehicle.position.longitude)
        : L.latLng(vehicle.latitude, vehicle.longitude);
    },
    inUse(vehicles) {
      return vehicles.filter(
        b => b.status === "IN_USE" && b.region === this.city
      );
    },
    available(vehicles) {
      return vehicles.filter(
        b => b.status === "AVAILABLE" && b.region === this.city
      );
    },
    adopted(vehicles) {
      return vehicles.filter(b => b.adopted && b.region === this.city);
    },
    getContent(b) {
      return JSON.stringify(b);
    },
    getIcon(type, status, adopted) {
      return L.icon({
        prefix: "",
        iconUrl: `static/${type}-${status}${adopted ? "-adopted" : ""}.png`,
        iconSize: type === "available" ? [10, 10] : [24, 36]
      });
    }
  },
  data() {
    return {
      map: null,
      city: "Angers",
      bicycles: [],
      scooters: [],
      zoom: 13,
      center: JSON.parse(JSON.stringify(cities["Angers"])),
      url:
        "https://api.mapbox.com/v4/mapbox.streets/{z}/{x}/{y}{r}.png?access_token=pk.eyJ1IjoicGllcnJpY2twIiwiYSI6ImNqZ21ldDRhMDFnZGczM3BhM2M4bXB2dmgifQ.sYk2UQOyEy0adoOXP_Rd7g",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    };
  },
  computed: {
    regions() {
      return [
        ...new Set([
          ...new Set(this.bicycles.map(b => b.region)),
          ...new Set(this.scooters.map(b => b.region))
        ])
      ];
    },
    bicyclesInUse() {
      return this.inUse(this.bicycles).length;
    },
    bicyclesAvailable() {
      return this.available(this.bicycles).length;
    },
    bicyclesAdopted() {
      return this.adopted(this.bicycles).length;
    },
    scootersInUse() {
      return this.inUse(this.scooters).length;
    },
    scootersAvailable() {
      return this.available(this.scooters).length;
    },
    scootersAdopted() {
      return this.adopted(this.scooters).length;
    }
  },
  firebase: {
    bicycles: db.ref("/rest/bicycles"),
    scooters: db.ref("/rest/scooters")
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

#controls {
  position: absolute;
  top: 0;
  right: 0;
  z-index: 10000;
}

.stats div {
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
