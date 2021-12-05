<template>
    <div class="flex flex-col h-screen max-h-screen">
        <!-- Search / Results -->
        <div class="flex justify-center bg-hero-pattern bg-cover relative px-4 pt-8 pb-32">
            <!-- Search Input -->
            <div class="w-full max-w-screen-sm">
                <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
                <div class="flex">
                    <input
                            v-model="queryIp"
                            type="text" class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
                            placeholder="Search for any IP address or leave empty to get your ip info"/>
                    <i @click="getIpInfo" class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fas fa-chevron-right"></i>
                </div>
            </div>
            <!-- IP Info -->
            <IPInfo class="z-20" v-if="ipInfo" :ipInfo="ipInfo" />
        </div>

        <!-- Map -->
        <div id="map" class="h-full z-10"></div>
    </div>
</template>

<script>
    import IPInfo from "@/components/IPInfo";
    import leaflet from "leaflet";
    import {onMounted, ref} from "@vue/runtime-core";
    import axios from "axios";
    import config from "../../config.json"

    export default {
        name: 'Home',
        components: {
            IPInfo,
        },
        setup() {
            let mMap;
            const queryIp = ref("");
            const ipInfo = ref(null);

            onMounted(() => {
                mMap = leaflet.map('map').setView([config.LATITUDE, config.LONGITUDE], 9);

                leaflet.tileLayer(`https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${config.MAP_BOX_ACCESS_TOKEN}`, {
                    attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
                    maxZoom: 18,
                    id: 'mapbox/streets-v11',
                    tileSize: 512,
                    zoomOffset: -1,
                    accessToken: config.MAP_BOX_ACCESS_TOKEN
                }).addTo(mMap);
            });

            const getIpInfo = async () => {

                try {
                    const data = await axios.get(`https://geo.ipify.org/api/v1?apiKey=${config.GEO_LOCATION_API_KEY}&ipAddress=${queryIp.value}`);
                    const result = data.data;

                    ipInfo.value = {
                        address: result.ip,
                        state: result.location.region,
                        timezone: result.location.timezone,
                        isp: result.isp,
                        latitude: result.location.lat,
                        longitude: result.location.lng,
                        name: result.as.name
                    };

                    leaflet.marker([ipInfo.value.latitude, ipInfo.value.longitude]).addTo(mMap);
                    mMap.setView([ipInfo.value.latitude, ipInfo.value.longitude], 13);
                }
                catch (e) {
                    alert(e.message);
                }

            };

            return {queryIp, ipInfo, getIpInfo};
        }
    }
</script>
