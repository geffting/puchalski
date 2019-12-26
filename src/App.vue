<template>
  <v-app id="main-content">
    <v-container>
      <v-row :class="[information ? '' : 'home']" align="center" no-gutters>
        <v-col>
          <v-row v-if="!information" no-gutters>
            <v-col cols="12">
              <span class="fs-35 text-center">Previsão do tempo</span>
            </v-col>
          </v-row>
          <v-row align="center" justify="center" class="pt-4" no-gutters>
            <v-col cols="12" sm="8">
              <v-text-field
                v-model="address"
                outlined
                label="Digite um local"
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
          <v-row class="mb-10" no-gutters>
            <v-col cols="12">
              <div>
                <span class="fs-100 text-center"><b> {{ temperature }}°C </b></span>
                <span class="fs-25 text-center"><b> {{ weather }} </b></span>
                <span class="fs-25 text-center"> {{ location }} </span>
                <span class="fs-25 text-center"> {{ currentDate }} </span>
              </div>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12">
              <v-row class="backblur" no-gutters>
                <v-col>
                  <span class="fs-20 text-center">Previsão</span>
                </v-col>
              </v-row>
              <v-row v-for="item in forecast" :key="item.id" align="center" class="backblur" no-gutters>
                <v-col cols="6">
                  <span class="fs-30 text-center"> {{ item.date }} </span>
                </v-col>
                <v-col cols="6">
                  <span class="fs-20 text-center"> {{ item.temperature }}°C<br> {{ item.weather }} </span>
                </v-col>
              </v-row>
            </v-col>
          </v-row>
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
    address: '',
    temperature: null,
    weather: null,
    location: null,
    currentDate: null,
    forecast: []
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
      const ref = this

      this.axios.get(`http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lng}&units=metric&lang=pt&appid=812f2be4009e5f5a153fcd3410a55a0a`)
        .then((response) => {
          ref.temperature = Math.round(response.data.main.temp)
          ref.weather = response.data.weather[0].description
          ref.location = response.data.name
          const today = new Date()
          const dd = String(today.getDate()).padStart(2, '0');
          const mm = String(today.getMonth() + 1).padStart(2, '0');
          ref.currentDate = dd + '/' + mm
          ref.information = true
        })
        .catch((error) => {
          alert(error)
        })

      this.axios.get(`http://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lng}&units=metric&lang=pt&appid=812f2be4009e5f5a153fcd3410a55a0a`)
        .then((response) => {
          ref.forecast = []

          const forecastList = response.data.list.filter( element => {
            return element.dt_txt.substring(11,13) === '12' // weather at 12h
          })

          let i = 0
          for(let item of forecastList){
            ref.forecast.push({
                                'id': i, 
                                'date': item.dt_txt.substring(8,10) + '/' + item.dt_txt.substring(5,7),
                                'weather': item.weather[0].description,
                                'temperature': Math.round(item.main.temp)
                              })
            i++
          }
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

span {
  display: block;
  color: #1e1559;
}

#main-content .home {
  height: 90vh;
}

.fs-20 {
  font-size: 20px;
}

.fs-25 {
  font-size: 25px;
}

.fs-30 {
  font-size: 30px;
}

.fs-35 {
  font-size: 35px;
}

.fs-100 {
  font-size: 100px;
}

#main-content .btn-search {
  margin: 0 0 30px 20px;
}

.backblur {
  background-color: rgba(255, 255, 255, 0.3);
  backdrop-filter: blur(10px);
  border: 1px solid transparent;
  margin: 5px;
  border-radius: 5px;
}

@media (max-width: 600px) {
  #main-content .btn-search {
    margin: 0 0 30px 0;
    width: 100%;
  }
}
</style>