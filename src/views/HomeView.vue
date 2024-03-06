<script setup>
import IPInfo from "@/components/IPInfo.vue";
import axios from "axios";
import leaflet from "leaflet";
import { onMounted, ref } from "vue";

let map;
const queryIp = ref("");
const ipInfo = ref(null);

onMounted(() => {
  map = leaflet.map("map").setView([51.505, -0.09], 13);

  leaflet
    .tileLayer(
      `https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${
        import.meta.env.VITE_API_LEAFLET
      }`,
      {
        attribution: '&copy; <a href="http://www.mapbox.com">MapBox</a>',
        maxZoom: 18,
        id: "mapbox/streets-v11",
        tileSize: 512,
        zoomOffset: -1,
        accessToken: import.meta.env.VITE_API_LEAFLET,
      }
    )
    .addTo(map);
});

const getIpInfo = async () => {
  try {
    const response = (
      await axios.get(
        `https://geo.ipify.org/api/v2/country?apiKey=${
          import.meta.env.VITE_API_IPGEO
        }&ipAddress=${queryIp.value}`
      )
    ).data;

    ipInfo.value = {
      address: response.ip,
      state: response.location.region,
      timezone: response.location.timezone,
      isp: response.isp,
      lat: response.location.lat,
      lng: response.location.lng,
    };
    const params = new URLSearchParams({
      access_token: import.meta.env.VITE_API_LEAFLET,
      fuzzyMatch: true,
      language: "en",
      limit: 1,
    });

    const location = (
      await axios(
        `https://api.mapbox.com/geocoding/v5/mapbox.places/${ipInfo.value.state}.json?${params}`
      )
    ).data;

    const newCoords = {
      lng: location.features[0].geometry.coordinates[0],
      lat: location.features[0].geometry.coordinates[1],
    };

    leaflet.marker([newCoords.lat, newCoords.lng]).addTo(map);
    map.setView([newCoords.lat, newCoords.lng], 8);
  } catch (error) {
    alert(error.message);
  }
};
</script>

<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- search / results -->
    <div
      class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32"
    >
      <!-- search input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input
            class="flex-1 py-3 pl-2 rounded-tl-md rounded-bl-md focus:outline-none"
            type="text"
            placeholder="Search for any IP address or leave it empty to get your IP info"
            v-model="queryIp"
          />
          <i
            class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fa-solid fa-chevron-right"
            @click="getIpInfo"
          ></i>
        </div>
      </div>
      <!-- ip info -->
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>
    <!-- map -->
    <div id="map" class="h-full z-10"></div>
  </div>
</template>
