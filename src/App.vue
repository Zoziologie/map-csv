<script setup>
const mapbox_access_token =
  "pk.eyJ1IjoicmFmbnVzcyIsImEiOiJjbGNvNmtteHQwNzBvM3JvNHhjYTJ3cmU3In0.i6uoI0lf5JfhzvKPSuGGjA";
const mapbox_layers = [
  {
    text: "Satelite",
    value: "satellite-v9",
  },
  {
    text: "Street",
    value: "streets-v11",
  },
  {
    text: "Outdoor",
    value: "outdoors-v9",
  },
  {
    text: "Satelite-Street",
    value: "satellite-streets-v11",
  },
  {
    text: "Light",
    value: "light-v10",
  },
  {
    text: "Dark",
    value: "dark-v10",
  },
];
</script>

<template>
  <div class="container-fluid h-100 d-flex flex-column">
    <div class="row flex-grow-1">
      <div class="col-md-2 vh-100 d-flex flex-column overflow-auto">
        <h3 class="mt-2">Map CSV</h3>
        <b-form-file @change="processFile" accept="csv" no-drop />
        <b-card class="mt-2 w-100" v-if="obs.length > 0" no-body>
          <b-card-body class="p-3">
            <b-form-group>
              <b-form-input lazy type="date" v-model="date_lim" debounce="1000"></b-form-input>
            </b-form-group>
            <b-input-group label="Between day of year:" class="mt-2">
              <b-form-input
                type="date"
                v-model="season_min"
                class="hide-year"
                debounce="1000"
              ></b-form-input>
              <b-form-input
                type="date"
                v-model="season_max"
                class="hide-year"
                debounce="1000"
              ></b-form-input>
            </b-input-group>
            <b-form-group class="mt-2 mb-0">
              <b-form-input type="number" v-model="count_lim" debounce="1000"></b-form-input>
            </b-form-group>
          </b-card-body>
        </b-card>
        <b-card class="mt-2" v-if="obs_info" no-body>
          <b-card-body class="p-3">
            <b>Species:</b
            ><a :href="'https://ebird.org/species/' + obs_info.species_code" target="_blank">
              {{ obs_info.species_code.toLocaleString() }} </a
            ><br />
            <b>Date:</b>
            {{
              new Date(obs_info.obs_dt).toLocaleDateString() +
              " " +
              new Date(obs_info.obs_dt).toLocaleTimeString().substring(0, 5)
            }}<br />
            <b>Checklist:</b>
            <a :href="'https://ebird.org/checklist/S' + obs_info.checklist_id" target="_blank">
              {{ obs_info.checklist_id }}
            </a>
            <br />
            <b>Count:</b> {{ obs_info.obs_count }}<br />
            <b>Duration:</b> {{ obs_info.effort_hrs }}<br />
            <b>Distance:</b> {{ obs_info.effort_distance_km }}
          </b-card-body>
        </b-card>
      </div>
      <div class="col flex-grow-1 px-0">
        <l-map
          class="w-100"
          style="height: 100%"
          :bounds="map_bounds"
          :options="{ zoomControl: false }"
        >
          <l-control-zoom position="bottomright"></l-control-zoom>
          <l-control position="topleft"> </l-control>

          <l-control-layers position="topright" />
          <l-tile-layer
            v-for="(l, id) in mapbox_layers"
            :key="l.text"
            :name="l.text"
            :visible="id == 1"
            :url="`https://api.mapbox.com/styles/v1/mapbox/${l.value}/tiles/{z}/{x}/{y}?access_token=${mapbox_access_token}`"
            layer-type="base"
          />

          <l-marker-cluster>
            <l-marker
              v-for="o in obs_filtered"
              :key="o.checklist_id"
              :lat-lng="[o.latitude, o.longitude]"
              @click="obs_info = o"
            >
            </l-marker>
          </l-marker-cluster>
        </l-map>
      </div>
    </div>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import "leaflet";
import "leaflet.markercluster/dist/MarkerCluster.css";
import "leaflet.markercluster/dist/MarkerCluster.Default.css";
import "./app.scss";

import {
  LMap,
  LTileLayer,
  LControlLayers,
  LControl,
  LPopup,
  LMarker,
  LIcon,
  LControlZoom,
} from "vue2-leaflet";
import Vue2LeafletMarkerCluster from "vue2-leaflet-markercluster";

export default {
  components: {
    LMap,
    LTileLayer,
    LControl,
    LControlLayers,
    LPopup,
    LMarker,
    LIcon,
    LControlZoom,
    "l-marker-cluster": Vue2LeafletMarkerCluster,
  },
  data() {
    return {
      obs: [],
      date_lim: "1990-01-01",
      season_min: "2020-01-01",
      season_max: "2020-12-31",
      count_lim: 0,
      map_bounds: null,
      obs_info: null,
    };
  },
  methods: {
    async onMapReady() {
      await this.$nextTick();
      this.map = this.$refs.map.mapObject;
    },
    processFile(event) {
      this.loading_file_status = 0;
      const file = event.target.files[0];

      const reader = new FileReader();
      reader.readAsText(file);
      reader.onerror = (error) => {
        throw new Error(error);
      };
      reader.onload = (e) => {
        this.obs = this.csvToArray(reader.result).map((o) => {
          o.doy = this.doy(o.obs_dt);
          o.obs_ms = new Date(o.obs_dt).getTime();
          o.obs_count = parseInt(o.obs_count);
          return o;
        });
        this.map_bounds = L.latLngBounds(this.obs.map((o) => L.latLng(o.latitude, o.longitude)));
      };
    },
    csvToArray(str, delimiter = ",") {
      const headers = str
        .slice(0, str.indexOf("\n"))
        .split(delimiter)
        .map((x) => x.replaceAll('"', ""));
      const rows = str
        .slice(str.indexOf("\n") + 1)
        .split("\n")
        .filter((row) => row.length > 0);
      return rows.map(function (row) {
        const values = row.split(delimiter).map((x) => x.replaceAll('"', ""));
        const el = headers.reduce(function (object, header, index) {
          object[header] = values[index];
          return object;
        }, {});
        return el;
      });
    },
    doy(date) {
      date = new Date(date);
      return (
        (Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()) -
          Date.UTC(date.getFullYear(), 0, 0)) /
        24 /
        60 /
        60 /
        1000
      );
    },
  },
  computed: {
    obs_filtered() {
      const date_lim = new Date(this.date_lim).getTime();
      const season_min = this.doy(this.season_min + "T00:00");
      const season_max = this.doy(this.season_max + "T00:00");
      console.log(date_lim);
      console.log(season_min);
      console.log(season_max);
      return this.obs.filter((o) => {
        return (
          o.obs_ms >= date_lim &&
          o.obs_count > this.count_lim &&
          o.doy >= season_min &&
          o.doy <= season_max
        );
      });
    },
  },
  mounted() {},
};
</script>
