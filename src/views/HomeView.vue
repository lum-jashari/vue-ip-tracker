<script setup>
import IPInfo from "@/components/IPInfo.vue";
import leaflet from "leaflet";
import { onMounted } from "vue";

let myMap;

onMounted(() => {
  myMap = leaflet.map("map").setView([51.505, -0.09], 13);

  leaflet
    .tileLayer(
      "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=",
      {
        attribution: '&copy; <a href="http://www.mapbox.com">MapBox</a>',
        maxZoom: 18,
        id: "mapbox/streets-v11",
        tileSize: 512,
        zoomOffset: -1,
        accessToken: "",
      }
    )
    .addTo(myMap);
});
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
          />
          <i
            class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fa-solid fa-chevron-right"
          ></i>
        </div>
      </div>
      <!-- ip info -->
      <IPInfo />
    </div>
    <!-- map -->
    <div id="map" class="h-full z-10"></div>
  </div>
</template>
