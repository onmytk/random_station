<script setup lang="ts">
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LMarker } from "@vue-leaflet/vue-leaflet";

const zoom = ref(9)
const center = [35.6769883, 139.7588499]
const coordinates = ref([50, 50])
const selectedPref = ref([14])
let roulette = ref(false)

// 駅データを取得
const { data: stationData } = await useFetch('/station/station20230327free.json', {
  transform: data => data.filter(p => {
    // 廃止されている駅は除外する
    return p.close_ymd === "0000-00-00" || new Date(p.close_ymd) > new Date()
  })
})

// 県データを取得
const { data: prefMaster } = await useFetch('/station/pref_master.json')

// 駅データから選択されている県
const randomSelect = () => {
  const filterdStation = stationData.value.filter(s => selectedPref.value.includes(s.pref_cd))
  return filterdStation[Math.floor(Math.random() * filterdStation.length)]
}

// 結果
let result = ref({})

// setInterval用のID
let nIntervalId = ref();

// ルーレット開始
const startRandomSelect = () => {
  nIntervalId = setInterval(() => {
    result.value = randomSelect()
  }, 100)
  roulette.value = true
}

// ルーレット停止
const stopRandomSelect = () => {
  clearInterval(nIntervalId)
  roulette.value = false
}

// URLの設定
const url = computed(() => {
  return "https://www.google.com/search?q="+result.value.station_name
})

const checkPrefList = computed(() => {
  return selectedPref.value.length < 0 ? false : true
})

watch(selectedPref, () => { console.log(selectedPref.value)})

</script>

<template>
  <!-- 対象の県の選択 -->
  <v-select
    v-model="selectedPref"
    label="Select"
    :items="prefMaster"
    item-title="pref"
    item-value="key"
    variant="outlined"
    clearable
    multiple
    chips
  >
  </v-select>

  <!-- スタート/ストップボタン -->
  <v-btn
    color="primary"
    @click="startRandomSelect"
    v-if="!roulette"
  >
    スタート
  </v-btn>

  <v-btn
    color="blue"
    @click="stopRandomSelect"
    v-if="roulette"
  >
    ストップ
  </v-btn>

  <!-- 駅名表示 -->
  <a v-if="result.station_name" :href="url" target="_blank" rel="noopener noreferrer">{{ result.station_name }}駅</a>

  <!-- 地図 -->
  <div style="height:80%; width:100%">
    <l-map :use-global-leaflet="false" ref="map" v-model:zoom="zoom" :center="center">
        <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
      ></l-tile-layer>

      <!-- マーカー表示 -->
      <l-marker :lat-lng="[result.lat ? result.lat : 0, result.lon ? result.lon : 0]"> </l-marker>
    </l-map>
  </div>
</template>
