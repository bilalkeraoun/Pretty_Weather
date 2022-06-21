<template>
    <!--FONT AWESOME-->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">

    <div class="container">
        <div class="overlay-icone-chargement" v-if="loading">
            <img src="./../../../assets/icons/circles.svg" alt="logo chargement">
        </div>


        <!-- TITLE -->
        <div class="title-flex">
            <img src="./../../../assets/icons/coollogo_com-614625.png" class="title-logo">
            <h1 class="title"></h1>
        </div>

        <!-- HEADER -->
        <div class="header_flex">
            <div class="header_item-flex">
             <img src="./../../../assets/icons/3d/6.png" alt="" class="header-logo-meteo">
            </div>
            <div class="item-flex">
                <p class="header_location">
                    <span id="location">{{ location.data.name }}</span>,
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
                    <img src="./../../../assets/icons/prochaines-heures.png" class="box-3d-img">
                    <div v-for="hour in next_hours" :key="hour">
                            <div class="box-margin">
                                {{ hour.date.getHours() }}h<br>
                                <img :src="require('./../../../assets/icons/' + hour.data.weather[0].icon + '.svg')" alt="logo du temps qu'il fait" class="box-icones"><br>
                                <strong>{{ Math.round(hour.data.temp) }}°C</strong><br>
                            </div>
                   </div>
                </div>
               
        </div>
          <!-- NEXT DAYS -->
                <div class="box">
                    <img src="./../../../assets/icons/prochaines-jours.png" class="box-3d-img">
                    <div class="box-margin" v-for="day in next_days" :key="day">
                           <span>{{ getFormatedDate(day.date) }}</span> <br>
                        <img :src="require('./../../../assets/icons/' + day.data.weather[0].icon + '.svg')" alt="logo du temps qu'il fait" class="box-icones"><br>
                        <p>
                           <strong>{{ Math.round(day.data.temp.min) }}°C | {{ Math.round(day.data.temp.max) }}°C</strong><br>
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
        background: linear-gradient(to bottom right, #5eb7b7, #5C9696);
    }
   

    /* TITLE  */
   .title-flex{
        display: flex;
        justify-content: center;
   }
    .title-logo{
        width: 40rem;
        padding: 1.5rem;
    }

    /* HEADER */
    .header_flex {
        display:         flex;
        justify-content: center;
        align-items:     center;
    }
    .header-logo-meteo {
        margin: 2rem;
        width: 50rem;
    }
    .header_temp {
    font-size: 6rem;
    }
    .header_location {
    font-size: 2.5rem;
    }
    #location {
        font-size: 3.5rem;
    }

    /* BOX */
    .box {
        text-align:    center;
        display:       flex;
        align-items:   center;
        font-size:     2rem;
        padding:       4rem 1rem 4rem 1rem;
    }
    .box-margin{
        font-size: 2rem;
        margin:    0 2rem;
    }
    .box-icones{
        width: 10rem;
    }
    .box-3d-img { 
        width: 20rem;
        padding: 0 7rem;
    }
    span { 
        font-size: 1.5rem;
    }


    .padding-hours {
      padding-right: 2rem;
    }
    .box-title-padding {
        padding-right: 3rem;
        font-size: 2rem;
    }

    .padding-days {
        padding-right: 2rem;
    }
    .linear-gradient {
        background: linear-gradient(to bottom right, #5eb7b7, #5C9696);
    }
    .prochaines-img {
        width: 16rem;
    }

    .next-hours_hours {
        font-size: 1.5rem;
        padding-left: 5rem;
    }
    .next-hours_deg {
        font-size: 2rem;
    }
    .bloc-logo-meteo {
        width: 5.5rem;
    }





    /* FOOTER */
    .footer {
        background:  rgb(249, 249, 249);
        padding:     3rem 0px;
        text-align:  center;
    }
    .footer .row {
        width:     100%;
        margin:    1% 0%;
        padding:   0.6% 0%;
        color:     rgb(45, 45, 45);
        font-size: 2rem;
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
