<template>
  <div id="map-container">
    <div id="map"></div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl';
import axios from 'axios';

const tokenMapBox = import.meta.env.VITE_TOKEN_MAPBOX;
const baseUrl = import.meta.env.VITE_BASE_URL;

export default {
  name: 'Map',
  async mounted() {
    try {
      mapboxgl.accessToken = tokenMapBox;
      this.map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/streets-v11',
        center: [37.6173, 55.7558],
        zoom: 4,
      });

      const response = await axios.get(baseUrl);
      const markers = response?.data?.data;

      markers.forEach(({ attributes }) => {
        new mapboxgl.Marker()
          .setLngLat(attributes.coordinates)
          .setPopup(new mapboxgl.Popup().setHTML(`<h3>${attributes.name}</h3>`))
          .addTo(this.map);
      });

      this.map.on('dblclick', async (e) => {
        const { lng, lat } = e.lngLat;
        const name = prompt('Введите название:');

        if (name) {
          new mapboxgl.Marker()
            .setLngLat({ lng, lat })
            .setPopup(new mapboxgl.Popup().setHTML(`<h3>${name}</h3>`))
            .addTo(this.map);

          await axios.post(baseUrl, {
            data: { coordinates: { lng, lat }, name },
          });
        }
      });
    } catch (error) {
      console.log(error);
    }
  },
};
</script>

<style>
#map-container {
  width: 800px;
  height: 800px;
  display: grid;
}

#map {
  grid-column: 1;
  grid-row: 1;
  width: 100%;
  height: 100%;
}

.mapboxgl-popup-content {
  padding: 20px;
  color: #000;
  font-size: 16px;
  text-transform: capitalize;
}
</style>
