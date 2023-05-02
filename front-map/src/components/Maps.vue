<template>
  <div id="map"></div>
</template>

<script>
import mapboxgl from 'mapbox-gl';
import axios from 'axios';

const tokenMapBox = import.meta.env.VITE_TOKEN_MAPBOX;
const baseUrl = import.meta.env.VITE_BASE_URL;

export default {
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
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [37.6173, 55.7558],
      zoom: 10,
    });
    this.map.on('load', () => {
      this.getPlots();
      this.addPolygon();
      this.removePolygon();
    });
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
          this.map.addSource(`plot-${plot.id}`, {
            type: 'geojson',
            data: {
              type: 'FeatureCollection',
              features: [polygon],
            },
          });
          this.map.addLayer({
            id: `plot-${plot.id}-layer`,
            type: 'fill',
            source: `plot-${plot.id}`,
            paint: {
              'fill-color': '#0080ff',
              'fill-opacity': 0.5,
            },
          });
          this.plots.push(plot);
        });
      } catch (e) {
        console.log(e);
      }
    },
    addPolygon() {
      this.map.on('dblclick', async (e) => {
        const isCreatePolygon = confirm('Create a new polygon?');
        const level = isCreatePolygon && prompt('Enter level:', 0);

        if (level) {
          const points = [];
          const clickHandler = async (e) => {
            const clickedPoint = [e.lngLat.lng, e.lngLat.lat];
            points.push(clickedPoint);

            this.map.addLayer({
              id: `point-${points.length}`,
              type: 'circle',
              source: {
                type: 'geojson',
                data: {
                  type: 'FeatureCollection',
                  features: [
                    {
                      type: 'Feature',
                      geometry: {
                        type: 'Point',
                        coordinates: clickedPoint,
                      },
                    },
                  ],
                },
              },
              paint: {
                'circle-radius': 5,
                'circle-color': '#0080ff',
              },
            });

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
                this.map.addSource(`plot-${newPlot.id}`, {
                  type: 'geojson',
                  data: {
                    type: 'FeatureCollection',
                    features: [polygon],
                  },
                });
                this.map.addLayer({
                  id: `plot-${newPlot.id}-layer`,
                  type: 'fill',
                  source: `plot-${newPlot.id}`,
                  paint: {
                    'fill-color': '#0080ff',
                    'fill-opacity': 0.5,
                  },
                });
                this.plots.push(newPlot);
              } catch (e) {
                console.log(e);
              }
              for (let i = 1; i <= 4; i++) {
                this.map.removeLayer(`point-${i}`);
              }
            }
          };
          this.map.on('click', clickHandler);
        }
      });
    },
    removePolygon() {
      this.map.on('contextmenu', async (e) => {
        const clickedFeature = this.map.queryRenderedFeatures(e.point, {
          layers: this.plots.map((plot) => `plot-${plot.id}-layer`),
        })[0];

        if (clickedFeature) {
          const id = clickedFeature.layer.id.split('-')[1];
          const isDeletePolygon = confirm(`Delete polygon?`);

          if (isDeletePolygon) {
            const plotIndex = this.plots.findIndex((plot) => plot.id == id);
            const plot = this.plots[plotIndex];
            this.plots.splice(plotIndex, 1);

            this.map.removeLayer(`plot-${plot.id}-layer`);

            try {
              await axios.delete(`${baseUrl}/${plot.id}`);
            } catch (e) {
              console.log(e);
            }
          }
        }
      });
    },
  },
};
</script>

<style>
#map {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 1200px;
}
</style>
