<script setup>
import { ref, watch } from 'vue';

const BASE_URL = 'https://psgc.gitlab.io/api';

let regions = ref([]);
let regionIsLoading = ref(false);
let provinces = ref([]);
let provinceIsLoading = ref(false);
let isNCRNote = ref('');
let municipalities = ref([]);
let municipalitiesIsLoading = ref(false);
let barangays = ref([]);
let barangaysIsLoading = ref(false);
let selectedRegion = ref('');
let selectedProvince = ref('');
let selectedMunicipality = ref('');
let selectedBarangay = ref('');

function _formatForVSelectApi(x, titleProp, valueProp) {
  let tmparr = [];
  x.forEach(function(o, i) {
    tmparr.push({'title':o[titleProp], 'value':o[valueProp]});
  });
  return tmparr;
}

async function getRegionsFromApi() {
  try {
    regionIsLoading.value = true;
    let response = await fetch(`${BASE_URL}/regions/`);
    regions.value = _formatForVSelectApi(await response.json(), 'regionName', 'code');
  } catch (error) {
    regionIsLoading.value = false;
    alert(`Error! Could not reach the API. ${error}`);
  } finally {
    regionIsLoading.value = false;
  }
}

getRegionsFromApi();

watch(selectedRegion, async() => {
  if (!selectedRegion.value) {
    provinces.value = [];
  } else {
    try {
      provinceIsLoading.value = true;
      selectedProvince.value = '';
      selectedMunicipality.value = '';
      selectedBarangay.value = '';
      let response;
      if (selectedRegion.value == '130000000') { //NCR
        provinces.value = [];
        isNCRNote.value = 'The National Capital Region (NCR) is the only region in the country without any province. It is subdivided into 17 local government units (LGUs) comprising of 16 cities and one municipality';
        response = await fetch(
          `${BASE_URL}/regions/${selectedRegion.value}/cities-municipalities`
        );
        municipalities.value = _formatForVSelectApi(await response.json(), 'name', 'code');
      } else {
        isNCRNote.value = '';
        response = await fetch(
        `${BASE_URL}/regions/${selectedRegion.value}/provinces`
        );
        provinces.value = _formatForVSelectApi(await response.json(), 'name', 'code');
      }
    } catch (error) {
      provinceIsLoading.value = false;
      alert(`Error! Could not reach the API. ${error}`);
    } finally {
      provinceIsLoading.value = false;
    }
  }
});

watch(selectedProvince, async() => {
  if (!selectedProvince.value) {
    municipalities.value = [];
  } else {
    try {
      municipalitiesIsLoading.value = true;
      selectedMunicipality.value = '';
      selectedBarangay.value = '';
      let response = await fetch(
        `${BASE_URL}/provinces/${selectedProvince.value}/cities-municipalities`
      )
      municipalities.value = _formatForVSelectApi(await response.json(), 'name', 'code');
    } catch (error) {
      municipalitiesIsLoading.value = false;
      alert(`Error! Could not reach the API. ${error}`);
    } finally {
      municipalitiesIsLoading.value = false;
    }
  }
});

watch(selectedMunicipality, async() => {
  if (!selectedMunicipality.value) {
    barangays.value = [];
  } else {
    try {
      barangaysIsLoading.value = true;
      selectedBarangay.value = '';
      let response = await fetch(
        `${BASE_URL}/cities-municipalities/${selectedMunicipality.value}/barangays`
      )
      barangays.value = _formatForVSelectApi(await response.json(), 'name', 'code');
    } catch (error) {
      barangaysIsLoading.value = false;
      alert(`Error! Could not reach the API. ${error}`);
    } finally {
      barangaysIsLoading.value = false;
    }
  }
});

</script>

<template>
  <v-container>
    <v-row>
      <v-col cols="12"><h1>&#129445;PSGC APP</h1></v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <small>Developed by Patrick Sabado visit my LinkedIn page at <a href="https://www.linkedin.com/in/wawawasabado/" target="_blank">https://www.linkedin.com/in/wawawasabado/</a></small>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12"><p>My first app using Vue.js. This app display Regions &rarr; Provinces &rarr; Cities/Municipalities &rarr; Barangays all over the Philippines powered by <a href="https://psgc.gitlab.io/api/" target="_blank">https://psgc.gitlab.io/api/</a></p></v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-select
          label="Choose a Region"
          :items="regions"
          v-model="selectedRegion"
          :loading="regionIsLoading"
          :disabled="regionIsLoading == true"
        ></v-select>
      </v-col>
      <v-col cols="12">
        <v-select
        label="Choose a Province"
        :items="provinces"
        :loading="provinceIsLoading"
        v-model="selectedProvince"
        :messages="isNCRNote"
        :disabled="isNCRNote != '' || provinceIsLoading == true"
        no-data-text="You must select a region first."
        ></v-select>
      </v-col>
      <v-col cols="12">
        <v-select
        label="Choose a City/Municipality"
        :items="municipalities"
        :loading="municipalitiesIsLoading"
        :disabled="municipalitiesIsLoading == true"
        v-model="selectedMunicipality"
        no-data-text="You must select a region and a province first."
        ></v-select>
      </v-col>
      <v-col cols="12">
        <v-select
        label="Choose a Barangay"
        :items="barangays"
        :loading="barangaysIsLoading"
        :disabled="barangaysIsLoading == true"
        v-model="selectedBarangay"
        no-data-text="You must select a region and a province and a city or municipality first."
        ></v-select>
      </v-col>
    </v-row>
  </v-container>
</template>
