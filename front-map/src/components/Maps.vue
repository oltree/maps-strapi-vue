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
  data() {
    return {
      map: null,
      plots: [],
    };
  },
  mounted() {
    mapboxgl.accessToken = tokenMapBox;
    this.map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [37.6173, 55.7558],
      zoom: 10,
    });
    this.getPlots();
    this.addContextMenuListener();
  },
  methods: {
    async getPlots() {
      try {
        const response = await axios.get(baseUrl);
        const plots = response?.data?.data;
        plots.forEach((plot) => {
          const polygon = {
            type: 'Feature',
            geometry: {
              type: 'Polygon',
              coordinates: [plot.attributes.points],
            },
            properties: { level: plot.attributes.level },
          };
          this.map.on('load', () => {
            this.map.addSource(`plot-${plot.attributes.level}`, {
              type: 'geojson',
              data: {
                type: 'FeatureCollection',
                features: [polygon],
              },
            });
            this.map.addLayer({
              id: `plot-${plot.id}-layer`,
              type: 'fill',
              source: `plot-${plot.attributes.level}`,
              paint: {
                'fill-color': '#0080ff',
                'fill-opacity': 0.5,
              },
            });
          });
          this.plots.push(plot);
        });
      } catch (error) {
        console.log(error);
      }
    },
    addContextMenuListener() {
      this.map.on('contextmenu', async (e) => {
        const level = prompt('Enter level:');
        if (level) {
          let points = [];
          const clickHandler = async (event) => {
            const clickedPoint = [event.lngLat.lng, event.lngLat.lat];
            points.push(clickedPoint);
            if (points.length === 4) {
              this.map.off('click', clickHandler);
              try {
                const response = await axios.post(baseUrl, {
                  data: { points, level },
                });
                const newPlot = response?.data?.data;
                const polygon = {
                  type: 'Feature',
                  geometry: {
                    type: 'Polygon',
                    coordinates: [newPlot.attributes.points],
                  },
                  properties: { level: newPlot.attributes.level },
                };
                this.plots.push(newPlot);
                this.map.addSource(`plot-${newPlot.attributes.level}`, {
                  type: 'geojson',
                  data: {
                    type: 'FeatureCollection',
                    features: [polygon],
                  },
                });
                this.map.addLayer({
                  id: `plot-${newPlot.id}-layer`,
                  type: 'fill',
                  source: `plot-${newPlot.attributes.level}`,
                  paint: {
                    'fill-color': '#0080ff',
                    'fill-opacity': 0.5,
                  },
                });
              } catch (error) {
                console.log(error);
              }
            }
          };
          this.map.on('click', clickHandler);
        }
      });
    },
  },
};
</script>

<style>
#map-container {
  width: 1280px;
  height: 100vh;
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
