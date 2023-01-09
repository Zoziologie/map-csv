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
      <l-control position="bottomright">
        <div class="d-flex flex-column" style="max-width: 480px">Test</div>
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
    </l-map>
  </div>
</template>

<script>
import "./app.scss";
import "leaflet/dist/leaflet.css";
import "leaflet";

import { LMap, LTileLayer, LControlLayers, LControl, LPopup, LMarker, LIcon } from "vue2-leaflet";

export default {
  components: {
    LMap,
    LTileLayer,
    LControl,
    LControlLayers,
    LPopup,
    LMarker,
    LIcon,
  },
  data() {
    return {
      map: null,
    };
  },
  methods: {
    async onMapReady() {
      await this.$nextTick();
      this.map = this.$refs.map.mapObject;
    },
  },
  mounted() {},
};
</script>
