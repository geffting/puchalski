<template>
  <v-app id="main-content">
    <v-container>
      <v-row :class="[temperature ? '' : 'home']" align="center" no-gutters>
        <v-col>
          <v-row v-if="!temperature" no-gutters>
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
                @click="getLocationInfo"
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
      <v-row v-if="temperature" no-gutters>
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
    address: '',
    temperature: null,
    weather: null,
    location: null,
    currentDate: null,
    forecast: []
  }),

  methods: {
    getLocationInfo () {
      const ref = this

      if (!ref.address || !ref.address.trim()) {
        return
      }

      let storedCoordinates = JSON.parse(localStorage.getItem('storedCoordinates'))
      let lat
      let lng

      if (storedCoordinates) {
        storedCoordinates.forEach((item) => {
          if (item.address == ref.address) {
            lat = item.lat
            lng = item.lng
          }
        })
      }

      if (lat && lng) {
        ref.getWeatherInfo(lat,lng)
      } else {
        this.axios.get(`https://maps.googleapis.com/maps/api/geocode/json?address=${ref.address}&key=AIzaSyCMNgEFKabcCJf2OPKLApy_FTxAWLTw--E`)
          .then((response) => {
            if (response.data.results.length === 0) {
              alert('Não encontrado')
              return
            }

            if (!storedCoordinates) {
              storedCoordinates = []
            }

            const geocode = {
              "address": ref.address,
              "lat": response.data.results[0].geometry.location.lat,
              "lng": response.data.results[0].geometry.location.lng
            }

            storedCoordinates.push(geocode)
            localStorage.setItem('storedCoordinates', JSON.stringify(storedCoordinates))

            ref.getWeatherInfo(response.data.results[0].geometry.location.lat,response.data.results[0].geometry.location.lng)
          })
          .catch((error) => {
            alert(error)
          })
      }
    },
    getWeatherInfo (lat,lng) {
      const ref = this

      const today = new Date()
      const dd = String(today.getDate()).padStart(2, '0')
      const mm = String(today.getMonth() + 1).padStart(2, '0')
      const year = String(today.getYear())
      ref.currentDate = dd + '/' + mm

      const currentHour = dd + '/' + mm + '/' + year + ' - ' + today.getHours()

      let storedWeather = JSON.parse(sessionStorage.getItem('storedWeather'))
      let findCacheWeather = false

      if (storedWeather) {
        storedWeather.forEach((item) => {
          if (item.lastQuery == currentHour && item.lat == lat && item.lng == lng) {
            ref.temperature = item.temperature
            ref.weather = item.weather
            ref.location = item.location
            findCacheWeather = true
          }
        })
      }

      if (findCacheWeather === false) {
        this.axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lng}&units=metric&lang=pt&appid=812f2be4009e5f5a153fcd3410a55a0a`)
          .then((response) => {
            ref.temperature = Math.round(response.data.main.temp)
            ref.weather = response.data.weather[0].description
            ref.location = response.data.name

            const weatherInfo = {
              "lastQuery": currentHour,
              "temperature": Math.round(response.data.main.temp),
              "weather": response.data.weather[0].description,
              "location": response.data.name,
              "lat": lat,
              "lng": lng
            }

            if (!storedWeather) {
              storedWeather = []
            }

            storedWeather.push(weatherInfo)
            sessionStorage.setItem('storedWeather', JSON.stringify(storedWeather))
          })
          .catch((error) => {
            alert(error)
          })
      }

      let storedForecast = JSON.parse(sessionStorage.getItem('storedForecast'))
      let findCacheForecast = false

      if (storedForecast) {
        storedForecast.forEach((item) => {
          if (item.lastQuery == currentHour && item.lat == lat && item.lng == lng) {
            ref.forecast = item.forecast
            findCacheForecast = true
          }
        })
      }

      if (findCacheForecast === false) {
        this.axios.get(`https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lng}&units=metric&lang=pt&appid=812f2be4009e5f5a153fcd3410a55a0a`)
          .then((response) => {
            const forecast = []

            const forecastList = response.data.list.filter( element => {
              return element.dt_txt.substring(11,13) === '12' // weather at 12h
            })

            let i = 0
            for (let item of forecastList) {
              forecast.push({
                              'id': i, 
                              'date': item.dt_txt.substring(8,10) + '/' + item.dt_txt.substring(5,7),
                              'weather': item.weather[0].description,
                              'temperature': Math.round(item.main.temp)
                            })
              i++
            }

            ref.forecast = forecast

            const forecastInfo = {
              "lastQuery": currentHour,
              "lat": lat,
              "lng": lng,
              "forecast": forecast
            }

            if (!storedForecast) {
              storedForecast = []
            }

            storedForecast.push(forecastInfo)
            sessionStorage.setItem('storedForecast', JSON.stringify(storedForecast))
          })
          .catch((error) => {
            alert(error)
          })
      }
    }
  }
}
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