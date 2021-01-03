<template>
  <div id="app">
    <div class="row no-gutters">
      <div class="col-sm-3">
        <div class="toolbox">
          <div class="sticky-top bg-white shadow-sm p-2">
            <div class="form-group d-flex">
              <label for="cityName" class="mr-2 col-form-label text-right"
                >縣市</label
              >
              <div class="flex-fill">
                <select id="cityName" class="form-control" v-model="select.city" @change="removeMarker(); updateMap();">
                  <option value="">-- Select One --</option>
                  <option v-for="c in cityName" :key="c.CityName" :value="c.CityName">
                    {{c.CityName}}
                  </option>
                </select>
              </div>
            </div>
            <div class="form-group d-flex">
              <label for="area" class="mr-2 col-form-label text-right"
                >地區</label
              >
              <div class="flex-fill">
                <select id="area" class="form-control">
                  <option value="">-- Select One --</option>
                </select>
              </div>
            </div>
            <p class="mb-0 small text-muted text-right">
              請先選擇區域查看（綠色表示還有口罩）
            </p>
          </div>
          <ul class="list-group">
            <template>
              <a class="list-group-item text-left">
                <h3>藥局名稱</h3>
                <p class="mb-0">成人口罩：1 | 兒童口罩：2</p>
                <p class="mb-0">
                  地址：<a
                    href="https://www.google.com.tw/maps/place/..."
                    target="_blank"
                    title="Google Map"
                  >
                    地址</a
                  >
                </p>
              </a>
            </template>
          </ul>
        </div>
      </div>
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet'
import cityName from './cityCountyData.json'

let osmMap

export default {
  name: 'App',
  data () {
    return {
      mapData: [],
      cityName,
      select: {
        city: '臺北市'
      }
    }
  },
  components: {

  },
  methods: {
    updateMap () {
      const pharmacies = this.mapData.filter(pharmacy =>
        pharmacy.properties.county === this.select.city
      )

      pharmacies.forEach(pharmacy => {
        const {
          properties: { name },
          geometry: { coordinates }
        } = pharmacy

        L.marker([coordinates[1], coordinates[0]]).addTo(osmMap).bindPopup(`藥局名稱 : ${name}`)
        this.penTo(pharmacies[0])
      })
    },
    removeMarker () {
      osmMap.eachLayer(layer => {
        if (layer instanceof L.Marker) {
          osmMap.removeLayer(layer)
        }
      })
    },
    penTo (pharmacy) {
      const {
        geometry: { coordinates }
      } = pharmacy

      osmMap.panTo([coordinates[1], coordinates[0]])
    }
  },
  created () {

  },
  mounted () {
    const url = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json'
    this.axios.get(url).then(res => {
      this.mapData = res.data.features
      this.updateMap()
    })

    osmMap = L.map('map', {
      center: [25.03, 121.55],
      zoom: 15
    })

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {}).addTo(osmMap)
  }
}
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap";

#map {
  height: 100vh;
}
.highlight {
  background: #e9ffe3;
}
.toolbox {
  height: 100vh;
  overflow-y: auto;
  a {
    cursor: pointer;
  }
}
</style>
