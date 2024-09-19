<template>
  <q-page >
  <div class="header mobile">
  <h4 class="text-white q-mb-lg">IP Address Tracker</h4>
  <q-input @keyup.enter="validateInput" v-model="input" :input-class="$q.screen.lt.sm ? 'q-ml-xs text-caption' : 'q-ml-sm text-subtitle1'" outlined color="black" bg-color="white" placeholder="Search for any IP address or domain"
  >
  <template v-slot:append>
          <q-btn
          @click="validateInput"
            flat
            round
            icon="chevron_right"
            class="black-btn"
          />
        </template>
  </q-input>
  </div>
<q-img height="300px" fit src="/images/pattern-bg-desktop.png">
   </q-img>
    <div id="map" style="height: 645px;"></div>
    <q-card flat bordered class="card mobile tablet">
<q-card-section :class="$q.screen.lt.sm ? 'column text-center' : 'row'">
<div :class="$q.screen.lt.md ? 'col q-ma-none' : 'col q-ma-md'">
<p :class="$q.screen.lt.sm ? 'gray-text text-caption q-my-none' : 'gray-text'">IP ADDRESS</p>
<p :class="$q.screen.lt.md ? 'text-h6 text-bold': 'text-h5 text-bold'">{{data.ipAddress }}</p>
</div>
<q-separator vertical></q-separator>
<div :class="$q.screen.lt.md ? 'col q-ma-none' : 'col q-ma-md'">
<p :class="$q.screen.lt.sm ? 'gray-text text-caption q-my-none' : 'gray-text'">LOCATION</p>
<p :class="$q.screen.lt.md ? 'text-h6 text-bold': 'text-h5 text-bold'">{{data.location}}</p>
</div>
<q-separator vertical></q-separator>
<div :class="$q.screen.lt.md ? 'col q-ma-none' : 'col q-ma-md'">
<p :class="$q.screen.lt.sm ? 'gray-text text-caption q-my-none' : 'gray-text'">TIMEZONE</p>
<p :class="$q.screen.lt.md ? 'text-h6 text-bold': 'text-h5 text-bold'">{{ data.timezone }}</p>
</div>
<q-separator vertical></q-separator>
<div :class="$q.screen.lt.md ? 'col q-ma-none' : 'col q-ma-md'">
<p :class="$q.screen.lt.sm ? 'gray-text text-caption q-my-none' : 'gray-text'">ISP</p>
<p :class="$q.screen.lt.md ? 'text-h6 text-bold': 'text-h5 text-bold'">{{data.isp}}</p>
</div>
</q-card-section>
</q-card>
  </q-page>
</template>

<script setup>
import { onMounted, reactive, ref} from 'vue';
import axios from 'axios';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import icon from '/images/icon-location.svg';
import iconShadow from 'leaflet/dist/images/marker-shadow.png';

const apiKey = 'at_HOmYU2cUClWC5g03vUFhSUprJby2i';
const map = ref(null);
const input = ref('');
const url = ref('');
const data = reactive({
  ipAddress: '',
  location: '',
  timezone: '',
  isp: '',
  domain: '',
  lat: '',
  lng: ''
})
const validateInput = () => {
  
  const isValidIPAddress = (input) => {
    const ipv4Regex =
      /^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
    const ipv6Regex =
      /(([0-9a-fA-F]{1,4}:){7}([0-9a-fA-F]{1,4}|:))|(::([0-9a-fA-F]{1,4}:){0,5}([0-9a-fA-F]{1,4}|:))/;
  
    if (ipv4Regex.test(input)) {
      return "IPv4";
    } else if (ipv6Regex.test(input)) {
      return "IPv6";
    } else {
      return false;
    }
  };
  
  const isValidDomain = (input) => {
    const domainRegex =
      /^(?!:\/\/)([a-zA-Z0-9-_]+\.)?[a-zA-Z0-9][a-zA-Z0-9-_]+\.[a-zA-Z]{2,11}?$/;
    return domainRegex.test(input);
  };
  
  const checkUserInput = (input) => {
    const ipAddressType = isValidIPAddress(input);
    
    if(ipAddressType) {
      url.value = `https://geo.ipify.org/api/v2/country,city?apiKey=${apiKey}&ipAddress=${input}`;
      fetchGeoData(url.value);
    }
    else if (isValidDomain(input)) {
      url.value = `https://geo.ipify.org/api/v2/country,city?apiKey=${apiKey}&domain=${input}`;
      fetchGeoData(url.value);
    } else {
      console.error("Invalid input format.");
      getDefaultLocation();
    }
  }

  checkUserInput(input.value);
  input.value = '';
}

const getDefaultLocation = () => {
  axios.get(`https://geo.ipify.org/api/v2/country,city?apiKey=${apiKey}`)
  .then(response => {
    data.ipAddress = response.data.ip;
    data.location = response.data.location.city + ','+ response.data.location.country + response.data.location.postalCode;
    data.timezone = response.data.location.timezone;
    data.isp = response.data.isp;
    data.domain = response.data.as.domain;
    data.lat = response.data.location.lat;
    data.lng = response.data.location.lng;
    updateMap(data.lat, data.lng); // Update map and add a marker to the default location
    console.log(response.data);
  }).catch(error => {
    console.log(error);
  })
}

const fetchGeoData = (urlType) => {
  axios.get(urlType)
  .then(response =>{
    data.ipAddress = response.data.ip;
    data.location = response.data.location.city + ','+ response.data.location.country;
    data.timezone = response.data.location.timezone;
    data.isp = response.data.isp;
    data.domain = response.data.as.domain;
    console.log(response.data);
    updateMap(response.data.location.lat, response.data.location.lng);
  })
  .catch(error => {
    console.log(error);
  })
}

const updateMap = (lat, lng) => {
  map.value.setView([lat, lng], 13); // Move map to the new location
  L.marker([lat, lng]).addTo(map.value); // Add a marker
  console.log(map.value) // Fetch geo data for the clicked location

};

onMounted(() => {
getDefaultLocation();
  map.value = L.map('map').setView([51.505, -0.09], 13);

  // Add the OpenStreetMap tile layer
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
  }).addTo(map.value);

  // Configure the marker with correct icon paths
  const defaultIcon = L.icon({
    iconUrl: icon,
    shadowUrl: iconShadow
  });
  L.Marker.prototype.options.icon = defaultIcon;
})
;

</script>
