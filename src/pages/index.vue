<script lang="ts" setup>
import { reactive, watch, onMounted } from "vue";
import axios from "axios";
//@ts-ignore
import _ from "lodash";

const BASE_URL = "https://a.868paga.com";

interface Item {
  id: string;
  name: string;
  short_name: string;
  icon_url: string;
  status: string;
  star: string;
  rtp_value: string;
  random: number;
  random_max: number;
  random_min: number;
  random_plus: number;
  expire_time: number;
}

interface Plataform {
  id: string;
  platform_name: string;
  icon_url: string;
  platform_id: string;
}

const state = reactive({
  data: [] as Item[],
  loading: true as boolean,
  error: null as any,
});

const plataformFilters = reactive({
  data: [] as Plataform[],
  filterPlataform: "1" as string,
});

const fetchPlataforms = () => {
  return axios
    .post<Plataform[]>(`${BASE_URL}/index.php/Index/index`)
    .catch((error) => {
      state.error = error;
      throw error; // Lançar o erro para ser capturado por .catch() no axios.all
    });
};

const fetchItems = () => {
  state.loading = true;
  return axios
    .post<Item[]>(
      `${BASE_URL}/index.php/Index/platform?platform_id=${plataformFilters.filterPlataform}`
    )
    .then((response) => {
      state.data = _.shuffle(response.data);
    })
    .catch((error) => {
      state.error = error;
      throw error; // Lançar o erro para ser capturado por .catch() no axios.all
    })
    .finally(() => {
      state.loading = false;
    });
};

onMounted(() => {
  axios
    //@ts-ignore
    .all([fetchPlataforms(), fetchItems()])
    .then(
      axios.spread((...responses) => {
        const plataformResponse = responses[0];
        const itemResponse = responses[1];
        plataformFilters.data = plataformResponse.data;
        //@ts-ignore
        state.data = _.shuffle(itemResponse.data);
      })
    )
    .catch((error) => {
      state.error = error;
    })
    .finally(() => {
      state.loading = false;
    });
});

watch(
  () => plataformFilters.filterPlataform,
  () => {
    fetchItems();
  }
);
</script>

<template>
  <v-container>
    <div
      style="position: fixed; inset: 0"
      class="d-flex justify-center align-center"
      v-if="state.loading"
    >
      <v-progress-circular
        color="secondary"
        :width="5"
        size="50"
        indeterminate
      ></v-progress-circular>
    </div>
    <div class="d-flex align-center justify-center py-12">
      <div class="d-inline-flex ga-5">
        <v-img
          v-for="item in plataformFilters.data"
          v-ripple
          class="rounded-circle"
          height="60"
          width="60"
          :key="item.id"
          :src="BASE_URL + item.icon_url"
          @click="plataformFilters.filterPlataform = item.platform_id"
          style="cursor: pointer"
        ></v-img>
      </div>
    </div>

    <v-row v-if="!state.loading">
      <v-col v-for="item in state.data" :key="item.id" cols="12" sm="4" md="3">
        <v-card class="mx-auto" max-width="400">
          <v-img class="align-end" width="100%" :src="BASE_URL + item.icon_url">
            <v-card-title>{{ item.name }}</v-card-title>
          </v-img>

          <v-card-subtitle class="pt-4">
            <v-rating
              v-model="item.star"
              color="amber"
              dense
              half-increments
              readonly
            ></v-rating>
          </v-card-subtitle>

          <v-card-text>
            <div class="d-flex ga-2 flex-column">
              <v-progress-linear
                color="grey-darken-1"
                height="15"
                :model-value="item.random"
                striped
              >
                <template v-slot:default="{ value }">
                  <strong> RTP por minuto : {{ Math.ceil(value) }}%</strong>
                </template>
              </v-progress-linear>
              <v-progress-linear
                color="blue"
                height="15"
                :model-value="item.random_min"
                striped
              >
                <template v-slot:default="{ value }">
                  <strong> Aposta mínima : {{ Math.ceil(value) }}%</strong>
                </template>
              </v-progress-linear>
              <v-progress-linear
                color="blue"
                height="15"
                :model-value="item.random_max"
                striped
              >
                <template v-slot:default="{ value }">
                  <strong> Aposta máxima : {{ Math.ceil(value) }}%</strong>
                </template>
              </v-progress-linear>
              <v-progress-linear
                color="green"
                height="15"
                :model-value="item.random_plus"
                striped
              >
                <template v-slot:default="{ value }">
                  <strong> Média diária: {{ Math.ceil(value) }}%</strong>
                </template>
              </v-progress-linear>
              <v-progress-linear
                color="green"
                height="15"
                :model-value="item.rtp_value"
                striped
              >
                <template v-slot:default="{ value }">
                  <strong> RTP ofical da PG: {{ Math.ceil(value) }}%</strong>
                </template>
              </v-progress-linear>
            </div>
          </v-card-text>

          <v-card-actions>
            <v-btn color="orange"> Jogar </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
