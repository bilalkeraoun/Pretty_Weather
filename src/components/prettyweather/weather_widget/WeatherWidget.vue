<template>
    <!--FONT AWESOME-->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">

    <div class="container">
        <div class="overlay-icone-chargement" v-if="loading">
            <img src="./../../../assets/icons/circles.svg" alt="logo chargement">
        </div>


        <!-- TITLE -->
        <div class="title">
            <h1>PRETTY WEATHER</h1>
        </div>

        <!-- HEADER -->
        <div class="header_flex">
            <div class="header_item-flex">
                <img :src="require('./../../../assets/icons/' + now.data.weather[0].icon + '.svg')" alt="logo du temps qu'il fait" class="header-logo-meteo">
                <!--<img src="./../../../assets/icons/3d/6.png" alt="" class="header-logo-meteo">-->
            </div>
            <div class="item-flex">
                <p class="header_location">
                    {{ location.data.name }}, 
                    {{ getFormatedDate(now.date) }} | {{ now.date.getHours() }}:{{ now.date.getMinutes() }}
                </p>
                <h2 class="header_temp">
                    {{ Math.round(now.data.temp) }}°C -
                    {{ now.data.weather[0].description}}
                </h2>
            </div>
        </div>

        <!-- BOX -->
        <div>
            <div class="linear-gradient">
                <!-- NEXT HOURS -->
                <div class="box">
                    <h5 class="box-title-padding">Prochaines Heures</h5>
                    <div class="padding-hours" v-for="hour in next_hours" :key="hour">
                            <div>
                                <span class="next-hours_hours">{{ hour.date.getHours() }}</span>h
                                <img :src="require('./../../../assets/icons/' + hour.data.weather[0].icon + '.svg')" alt="logo du temps qu'il fait" class="bloc-logo-meteo">
                                <strong>{{ Math.round(hour.data.temp) }}°C</strong>
                            </div>
                   </div>
                </div>
               
        </div>
          <!-- NEXT DAYS -->
                <div class="boxe">
                    <h5 class="box-title-padding">Prochain Jours</h5>
                    <div class="padding-days"  v-for="day in next_days" :key="day">
                        <p class="next-hours_hours">
                            {{ getFormatedDate(day.date) }}
                        </p>
                        <img :src="require('./../../../assets/icons/' + day.data.weather[0].icon + '.svg')" alt="logo du temps qu'il fait" class="bloc-logo-meteo">
                        <p>
                           <strong>{{ Math.round(day.data.temp.min) }}°C | {{ Math.round(day.data.temp.max) }}°C </strong> 
                        </p>
                    </div>
                    </div>
                </div>

        <!-- FOOTER -->
        <footer>
            <div class="footer">
            
            <div class="row">
             Pretty Weather Copyright © 2022 - All rights reserved || Designed By : Bilal.ux 
            </div>
            </div>
        </footer>
       


    </div>
</template>

<script>
export default {
    data(){
        return{
            loading: true,
            api_key: 'aa27e59d7dc0fcd71fbee699a2601225',
            location: {coords: {}, data: {}},
            full_weather: {},
            now: {date: null, data: {}},
            next_hours: [],
            next_days: []
        }
    },
    async created(){
        navigator.geolocation.getCurrentPosition(async(position) => {
            this.location.coords = await position.coords;
            this.getCurrentLocation();
            this.getFullWeather();
        });
    },
    methods: {
        async getCurrentLocation(){
            let location = await fetch(`http://api.openweathermap.org/geo/1.0/reverse?lat=${this.location.coords.latitude}&lon=${this.location.coords.longitude}&appid=${this.api_key}`);
            location = await location.json();
            this.location.data = location[0];
        },
        async getFullWeather(){
            let full_weather = await fetch(`https://api.openweathermap.org/data/2.5/onecall?lat=${this.location.coords.latitude}&lon=${this.location.coords.longitude}&exclude=minutely,alerts&units=metric&lang=fr&appid=${this.api_key}`);
            this.full_weather = await full_weather.json();
            this.loading = false;

            this.setNow();
            this.setNextHours();
            this.setNextDays();

            this.resetFullWeather();
        },
        resetFullWeather(){
            setInterval(() => { 
                // this.getFullWeather();
                }, 60000); 
        },
        setNow() {
            this.now = {
                date : new Date(this.full_weather.current.dt*1000),
                data : this.full_weather.current
            }
        },
         setNextHours () {
             this.next_hours = [];
            for (let i = 3; i < 22; i = i + 3) {
                this.next_hours.push({
                    date: new Date(this.full_weather.hourly[i].dt * 1000),
                    data: this.full_weather.hourly[i]
                });
            }
        },
        setNextDays () {
            for (let i = 1; i < 8; i++) {
                this.next_days.push({
                    date: new Date(this.full_weather.daily[i].dt * 1000),
                    data: this.full_weather.daily[i]
                });
            }
        },
        getFormatedDate(date) {
            return new Intl.DateTimeFormat('fr-FR',{weekday: 'long', day:'2-digit', month: "long"}).format(date)
        }
    }
}
</script>

<style>
    /* DEFAULT */
    body {
        font-family: Arial, Helvetica, sans-serif;
        color:       #f1f1f1;
        background:  rgb(48, 48, 48);
    }
    .container {
        background: linear-gradient(to bottom right, #5eb7b7, #5C9696);
        border:     1px solid #f1f1f1;
        width:      1100px;
        margin:     100px auto 0;
    }

    /* TITLE */
    .title h1 {
      text-align:            center;
      font-size:             2rem; 
      font-family:           'Calvier';
      padding-bottom:        1.5rem;
      border-bottom:         1.5px solid;
    }
    

    /* HEADER */
    .header_flex {
    display:         flex;
    justify-content: center;
    align-items:     center;
    margin: 0rem;
    }
    .header-logo-meteo {
    width: 40rem;
    }
    .header_temp {
    font-size: 3.5rem;
    }
    .header_location {
    font-size: 1.5rem;
    }


    /* NEXT HOURS | DAYS */
    .box {
        text-align:    center;
        display:       flex;
        align-items:   center;
        font-size:     1.2rem;
        padding:       1rem;
        padding-top:   4rem;
        padding-bottom: 4rem;
    }
    .boxe {
        text-align:    center;
        display:       flex;
        align-items:   center;
        font-size:     1.2rem;
        padding:       1rem;
        padding-top:   2rem;
        padding-bottom: 2rem;
    }

    .padding-hours {
      padding-right: 2rem;
    }
    .box-title-padding {
        padding-right: 3rem;
        font-size: 1.5rem;
    }

    .padding-days {
      padding-right: 2rem;
    }
    .linear-gradient {
        background: linear-gradient(to bottom right, #5eb7b7, #5C9696);
    }

    .next-hours_hours {
    font-size: 1.1rem;
    }
    .next-hours_deg {
    font-size: 1.3rem;
    }
    .bloc-logo-meteo {
    width: 5rem;
    }

    /* FOOTER */
    .footer {
        background:  rgb(249, 249, 249);
        padding:     5px 0px;
        text-align:  center;
    }
    .footer .row {
        width:     100%;
        margin:    1% 0%;
        padding:   0.6% 0%;
        color:     gray;
        font-size: 0.8em;
    }
    
    /* Animation chargement */
    .overlay-icone-chargement {
    position: absolute;
    width: 100%;
    height: 100%;
    background: linear-gradient(45deg, rgb(59,50,50), rgb(22,28,29));
    transition: opacity 1.1s ease-out;
    z-index: 1000;
    }
    .overlay-icone-chargement img {
    width: 150px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    }
    .disparition {
    opacity: 0;
    }
</style>
