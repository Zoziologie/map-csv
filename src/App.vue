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
  <div class="container-fluid h-100 d-flex p-0">
    <l-map
      class="w-100"
      style="height: 100%"
      ref="map"
      @ready="onMapReady"
      :zoom="1"
      :center="[0, 0]"
    >
      <l-control position="topleft">
        <b-form-file
          size="lg"
          @change="processFile"
          accept="csv"
          no-drop
          style="min-width: 200px"
        />
      </l-control>

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
        <l-marker v-for="o in obs" :key="o.checklist_id" :lat-lng="[o.latitude, o.longitude]">
          <l-popup>
            <b>Date:</b> {{ o.obs_dt }}<br />
            <b>Checklist:</b>
            <a :href="'https://ebird.org/checklist/' + o.checklist_id" target="_blank">
              {{ o.checklist_id }}
            </a>
            <br />
            <b>Count:</b> {{ o.obs_count }}<br />
            <b>Duration:</b> {{ o.effort_hrs }}<br />
            <b>Distance:</b> {{ o.effort_distance_km }}<br />
            <b>Species:</b>
            <a :href="'https://ebird.org/species/' + o.species_code" target="_blank">
              {{ o.species_code }}
            </a>
            <br />
          </l-popup>
        </l-marker>
      </l-marker-cluster>
    </l-map>
  </div>
</template>

<script>
import "./app.scss";
import "leaflet/dist/leaflet.css";
import "leaflet";
import "leaflet.markercluster/dist/MarkerCluster.css";
import "leaflet.markercluster/dist/MarkerCluster.Default.css";

import { LMap, LTileLayer, LControlLayers, LControl, LPopup, LMarker, LIcon } from "vue2-leaflet";
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
    "l-marker-cluster": Vue2LeafletMarkerCluster,
  },
  data() {
    return {
      obs: [],
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
        this.obs = this.csvToArray(reader.result);
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
  },
  mounted() {},
};
</script>
