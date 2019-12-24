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
                v-model="address"
                outlined
                label="Digite o endereço"
                color="#1e1559"
              />
            </v-col>
            <v-col cols="12" sm="2">
              <v-btn
                @click="getCoordinates"
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
    address: ''
  }),

  methods: {
    getCoordinates () {
      const ref = this

      if (!ref.address) {
        return
      }

      this.axios.get(`https://maps.googleapis.com/maps/api/geocode/json?address=${ref.address}&key=AIzaSyCMNgEFKabcCJf2OPKLApy_FTxAWLTw--E`)
        .then((response) => {
          if (response.data.results.length === 0) {
            alert('Não encontrado')
            return
          }

          ref.getWeatherInfo(response.data.results[0].geometry.location.lat,response.data.results[0].geometry.location.lng)
        })
        .catch((error) => {
          alert(error)
        })
    },

    getWeatherInfo (lat,lng) {
      this.axios.get(`http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lng}&units=metric&lang=pt&appid=812f2be4009e5f5a153fcd3410a55a0a`)
        .then((response) => {
          console.log('response.data: ', response.data) // eslint-disable-line no-console
        })
        .catch((error) => {
          alert(error)
        })

      this.axios.get()
        .then((response) => {
          console.log(response) // eslint-disable-line no-console
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