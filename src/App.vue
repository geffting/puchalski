<template>
  <v-app id="main-content">
    <v-container>
      <v-row :class="[temperature ? '' : 'home']" align="center" no-gutters>
        <v-col>
          <v-row v-if="!temperature" no-gutters>
            <v-col cols="12">
              <span class="fs-35 block text-center">Previsão do tempo</span>
            </v-col>
          </v-row>
          <v-row align="center" justify="center" class="pt-4" no-gutters>
            <v-col cols="12" sm="8">
              <v-combobox
                ref="searchCombobox"
                v-model="addressDummyModel"
                @change="onAutoCompleteSelection"
                @keyup="customOnChangeHandler"
                @paste="customOnChangeHandler"
                :items="items"
                label="Digite um local"
                color="#1e1559"
                outlined
              ></v-combobox>
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
                <span class="fs-100 block text-center"><b> {{ temperature }}°C </b></span>
                <span class="fs-25 block text-center"><b> {{ weather }} </b></span>
                <span class="fs-25 block text-center"> {{ location }} </span>
                <span class="fs-25 block text-center"> {{ currentDate }} </span>
              </div>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12">
              <v-row class="backblur" no-gutters>
                <v-col>
                  <span class="fs-20 block text-center">Previsão</span>
                </v-col>
              </v-row>
              <v-row v-for="item in forecast" :key="item.id" align="center" class="backblur" no-gutters>
                <v-col cols="6">
                  <span class="fs-30 block text-center"> {{ item.date }} </span>
                </v-col>
                <v-col cols="6">
                  <span class="fs-20 block text-center"> {{ item.temperature }}°C<br> {{ item.weather }} </span>
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
    address: null,
    addressDummyModel: null,
    temperature: null,
    weather: null,
    location: null,
    currentDate: null,
    forecast: [],
    comboBoxModel: null,
    items: []
  }),

  mounted () {
    let storedSearches = JSON.parse(localStorage.getItem('storedSearches'))

    if (storedSearches) {
      this.items = storedSearches
    }
  },

  methods: {
    onAutoCompleteSelection () {
      this.address = this.addressDummyModel
    },
    customOnChangeHandler () {
      setTimeout (() => {
        if (this.$refs.searchCombobox) {
          this.address = this.$refs.searchCombobox.internalSearch
        }
      });
    },
    async getLocationInfo () {
      if (!this.address || !this.address.trim()) {
        return
      }

      let storeAddress = true

      this.items.forEach((item) => {
        if (item == this.address) {
          storeAddress = false
        }
      })

      if (storeAddress === true) {
        this.items.push(this.address)
        localStorage.setItem('storedSearches', JSON.stringify(this.items))
      }

      let storedCoordinates = JSON.parse(localStorage.getItem('storedCoordinates'))
      let lat
      let lng

      if (storedCoordinates) {
        storedCoordinates.forEach((item) => {
          if (item.address == this.address) {
            lat = item.lat
            lng = item.lng
          }
        })
      }

      if (lat && lng) {
        this.getWeatherInfo(lat,lng)
      } else {
        const response = await this.axios.get(`https://maps.googleapis.com/maps/api/geocode/json?address=${this.address}&key=AIzaSyCMNgEFKabcCJf2OPKLApy_FTxAWLTw--E`)
        if (response.data.results.length === 0) {
          alert('Não encontrado')
          return
        }

        if (!storedCoordinates) {
          storedCoordinates = []
        }

        const geocode = {
          "address": this.address,
          "lat": response.data.results[0].geometry.location.lat,
          "lng": response.data.results[0].geometry.location.lng
        }

        storedCoordinates.push(geocode)
        localStorage.setItem('storedCoordinates', JSON.stringify(storedCoordinates))

        this.getWeatherInfo(response.data.results[0].geometry.location.lat,response.data.results[0].geometry.location.lng)
      }
    },
    async getWeatherInfo (lat,lng) {
      const today = new Date()
      const dd = String(today.getDate()).padStart(2, '0')
      const mm = String(today.getMonth() + 1).padStart(2, '0')
      const year = String(today.getYear())
      this.currentDate = dd + '/' + mm

      const currentHour = dd + '/' + mm + '/' + year + ' - ' + today.getHours()

      let storedWeather = JSON.parse(sessionStorage.getItem('storedWeather'))
      let findCacheWeather = false

      if (storedWeather) {
        storedWeather.forEach((item) => {
          if (item.lastQuery == currentHour && item.lat == lat && item.lng == lng) {
            this.temperature = item.temperature
            this.weather = item.weather
            this.location = item.location
            findCacheWeather = true
          }
        })
      }

      if (findCacheWeather === false) {
        const response = await this.axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lng}&units=metric&lang=pt&appid=812f2be4009e5f5a153fcd3410a55a0a`)
        this.temperature = Math.round(response.data.main.temp)
        this.weather = response.data.weather[0].description
        this.location = response.data.name

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
      }

      let storedForecast = JSON.parse(sessionStorage.getItem('storedForecast'))
      let findCacheForecast = false

      if (storedForecast) {
        storedForecast.forEach((item) => {
          if (item.lastQuery == currentHour && item.lat == lat && item.lng == lng) {
            this.forecast = item.forecast
            findCacheForecast = true
          }
        })
      }

      if (findCacheForecast === false) {
        const response = await this.axios.get(`https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lng}&units=metric&lang=pt&appid=812f2be4009e5f5a153fcd3410a55a0a`)
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

        this.forecast = forecast

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

.block {
  display: block;
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