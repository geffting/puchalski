<template>
  <v-app id="main-content">
    <v-container>
      <v-row :class="[information ? '' : 'home']" align="center" no-gutters>
        <v-col>
          <v-row v-if="!information" no-gutters>
            <v-col cols="12">
              <span class="header text-center">Previsão do tempo</span>
            </v-col>
          </v-row>
          <v-row align="center" justify="center" class="pt-4" no-gutters>
            <v-col cols="12" sm="8">
              <v-text-field
                v-model="userLocation"
                outlined
                label="Digite o endereço"
                color="#1e1559"
              />
            </v-col>
            <v-col cols="12" sm="2">
              <v-btn
                @click="getInfo"
                class="btn-search"
                outlined
                x-large
                elevation="2"
                color="#1e1559"
              >
                Buscar
              </v-btn>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <v-row v-if="information" no-gutters>
        <v-col cols="12">
          <div>
            <span class="temperature-info text-center">28°C</span>
            <span class="weather-info text-center">Nublado</span>
            <span class="city-info text-center">Joinville - Santa Catarina</span>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
export default {
  name: 'App',

  data: () => ({
    information: false,
    userLocation: null
  }),

  methods: {
    getInfo () {
      this.information = true

      this.axios.get(``)
        .then((response) => {
          console.log('google response: ', response) // eslint-disable-line no-console
        })
        .catch((error) => {
          alert(error)
        })
    }
  }
};
</script>
<style type="text/css">
#main-content {
  background: url('./assets/background.png') no-repeat center center/cover;
  font-family: 'Work Sans', sans-serif;
}

#main-content .home {
  height: 90vh;
}

#main-content .header {
  display: block;
  font-size: 35px;
  color: #1e1559;
}

#main-content .temperature-info {
  display: block;
  font-size: 100px;
  font-weight: bold;
  color: #1e1559;
}

#main-content .weather-info {
  display: block;
  font-size: 25px;
  font-weight: bold;
  color: #1e1559;
}

#main-content .city-info {
  display: block;
  font-size: 25px;
  color: #1e1559;
}

#main-content .btn-search {
  margin: 0 0 30px 20px;
}

@media (max-width: 600px) {
  #main-content .btn-search {
    margin: 0 0 30px 0;
    width: 100%;
  }
}
</style>