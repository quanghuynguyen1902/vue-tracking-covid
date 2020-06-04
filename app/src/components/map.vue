<template>
  <div class="wrapper">
    <l-map style="height:70vh" :zoom="zoom" @update:zoom="zoomUpdated" :center="center">
      <l-tile-layer :url="url"></l-tile-layer>
      <div class="marker">
        <div v-if="zoom > 0 && zoom < 5">
          <l-circle-marker
            v-for="location in countries_location"
            :key="location.location"
            fillColor="red"
            color="red"
            :radius="location.radius"
            :fillOpacity="0.5"
            :lat-lng="[location.latitude, location.longitude]"
          >
            <l-popup class="tagInfor">
              <div id="location">{{location.location}}</div>
              <div id="confirmed">Confirmed: {{location.confirmed}}</div>
              <div id="dead">Dead: {{location.dead}}</div>
              <div id="recovered">Recovered: {{location.recovered}}</div>
            </l-popup>
          </l-circle-marker>
        </div>
        <div v-if="zoom >= 5 && zoom < 6">
          <l-circle-marker
            v-for="location in provinces_location"
            :key="location.location"
            fillColor="red"
            color="red"
            :radius="5"
            :fillOpacity="1"
            :lat-lng="[location.latitude, location.longitude]"
          >
            <l-popup class="tagInfor">
              <div id="location">{{location.location}}</div>
              <div id="confirmed">Confirmed: {{location.confirmed}}</div>
              <div id="dead">Dead: {{location.dead}}</div>
              <div id="recovered">Recovered: {{location.recovered}}</div>
            </l-popup>
          </l-circle-marker>
        </div>
        <div v-if="zoom >= 6">
          <l-circle-marker
            v-for="(location, index) in cities_location"
            :key="index"
            fillColor="red"
            color="white"
            :radius="3"
            :fillOpacity="1"
            :lat-lng="[location.latitude, location.longitude]"
          >
            <l-popup class="tagInfor">
              <div id="location">{{location.location}}</div>
              <div id="confirmed">Confirmed: {{location.confirmed}}</div>
              <div id="dead">Dead: {{location.dead}}</div>
              <div id="recovered">Recovered: {{location.recovered}}</div>
            </l-popup>
          </l-circle-marker>
        </div>
      </div>
      <l-geo-json :geojson="countryGeoJson" :options="options" :options-style="styleFunction"></l-geo-json>
    </l-map>
    <div class="infor align-items-center" >
      <div class="row justify-content-center">
        <div class="col-md-2" id="infor-confirmed-number">{{this.total.confirmed.toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,')}}</div>
        <div class="col-md-2" id="infor-dead-number">{{this.total.dead.toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,')}}</div>
        <div class="col-md-2" id="infor-recovered-number">{{this.total.recovered.toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,')}}</div>
      </div>
      <div class="row justify-content-center">
        <div class="col-md-2" id="infor-confirmed-text">Confirmed</div>
        <div class="col-md-2" id="infor-dead-text">Dead</div>
        <div class="col-md-2" id="infor-recovered-text">Recovered</div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  LMap,
  LTileLayer,
  LCircleMarker,
  LPopup,
  LGeoJson
} from "vue2-leaflet";
import countriesGeoJson from "../data/countries.json";
import axios from "axios";
export default {
  components: {
    LMap,
    LTileLayer,
    LCircleMarker,
    LPopup,
    LGeoJson
  },
  data() {
    return {
      url:
        "https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png",
      zoom: 3,
      center: [9.779349, 105.6189045, 11],
      countries_location: [],
      provinces_location: [],
      cities_location: [],
      total: {
        confirmed: 0,
        dead: 0,
        recovered: 0
      },
      color: "#000000",
      countryGeoJson: countriesGeoJson
    };
  },
  mounted() {
    this.add_Radius();
    this.get_provinces();
    this.get_cities();
  },
  computed: {
    options() {
      return {
        onEachFeature: this.onEachFeatureFunction
      };
    },
    styleFunction() {
      return () => {
        return {
          weight: 1.5,
          color: this.color,
          opacity: 0.5,
          fillOpacity: 0
        };
      };
    }
  },
  methods: {
    zoomUpdated(zoom) {
      this.zoom = zoom;
    },
    get_provinces() {
      const path = "https://www.trackcorona.live/api/provinces";
      axios.get(path).then(res => {
        this.provinces_location = res.data.data;
      });
    },
    async get_cities() {
      let response = await fetch("https://www.trackcorona.live/api/cities");
      let data = await response.json();
      this.cities_location = await data.data;
    },
    async add_Radius() {
      let response = await fetch("https://www.trackcorona.live/api/countries");
      let data = await response.json();
      this.countries_location = await data.data;

      for (let i in this.countries_location) {
        this.total.confirmed += this.countries_location[i].confirmed;
        this.total.dead += this.countries_location[i].dead;
        this.total.recovered += this.countries_location[i].recovered;
        if (this.countries_location[i].confirmed < 1000) {
          this.countries_location[i]["radius"] = 8;
        } else if (this.countries_location[i].confirmed < 5000) {
          this.countries_location[i]["radius"] = 10;
        } else if (this.countries_location[i].confirmed < 10000) {
          this.countries_location[i]["radius"] = 15;
        } else if (this.countries_location[i].confirmed < 30000) {
          this.countries_location[i]["radius"] = 20;
        } else if (this.countries_location[i].confirmed < 50000) {
          this.countries_location[i]["radius"] = 25;
        } else if (this.countries_location[i].confirmed < 1000000) {
          this.countries_location[i]["radius"] = 35;
        } else if (this.countries_location[i].confirmed < 10000000) {
          this.countries_location[i]["radius"] = 40;
        } else {
          this.countries_location[i]["radius"] = 0;
        }
      }
    },
    highlightFeature(e) {
      var layer = e.target;
      layer.setStyle({
        color: "#ffffff"
      });
    },
    resetFeature(e) {
      var layer = e.target;
      layer.setStyle({
        color: "#000000"
      });
    },
    onEachFeatureFunction(feature, layer) {
      layer.on({
        mouseover: this.highlightFeature,
        mouseout: this.resetFeature
      });
    }
  }
};
</script>

<style>
.wrapper {
  width: 100%;
  height: 100%;
  background-color: #000000;
}

.tagInfor #location {
  color: #000000;
  font-size: 20px;
}

.tagInfor #confirmed {
  color: #e67300;
}
.tagInfor #dead {
  color: #ff0000;
}
.tagInfor #recovered {
  color: #00b300;
}
.infor {
  height: 30vh;
}

#infor-confirmed-number {
  font-size: 50px;
  color: #e67300;
}
#infor-confirmed-text {
  left: 50px;
  font-size: 24px;
  color: rgb(155, 154, 154);
}

#infor-dead-number {
  font-size: 50px;
  color: #ff0000;
}
#infor-dead-text {
  left: 50px;
  font-size: 24px;
  color: rgb(155, 154, 154);
}

#infor-recovered-number {
  font-size: 50px;
  color: #00b300;
}
#infor-recovered-text {
  left: 50px;
  font-size: 24px;
  color: rgb(155, 154, 154);
}

</style>