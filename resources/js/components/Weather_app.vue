<template>
    <div class="text-white mb-8">
        <div class="places-input text-gray-800">
            <input type="search" id="city" class="w-full form-control" placeholder="Kuriame mieste gyveni?" />
            <p>Pasirinkta: <strong id="address-value">kol kas nieko.. :)</strong></p>
        </div>

        <div class="weather-container font-sans md:w-128 max-w-lg rounded-lg overflow-hidden bg-gray-900 shadow-lg mt-8">
            <div class="current-weather flex items-center justify-between px-6 py-4">
                <div class="flex items-center">
                    <div>
                        <div class="text-5xl font-semibold"> {{ currentTemperature.actual}} °C</div>
                        <div>Pojutis kaip {{ currentTemperature.feels }}°C</div>
                    </div>
                    <div class="mx-5">
                        <div class="font-semibold"> {{ currentTemperature.summary }} </div>
                        <div> {{ location.name }} </div>
                    </div>
                </div>
                <div><img :src="'icons/'+currentTemperature.icon+'.png'" alt="icon"></div>
            </div> <!-- end current-weather -->

        <div class="future-weather text-sm bg-gray-800 px-6 py-4 overflow-hidden">
            <div
                v-for="(day, index) in daily"
                :key=day.dt
                class="flex items-center"
                :class="{'mt-8' : index > 0}"
                v-if="index==8 || index==16 || index==24 || index==32 || index==40"
                >
                <div class="w-1/6 text-lg text-gray-200">{{toDayOfWeek(day.dt)}}</div>
                <div class="w-4/6 px-4 flex items-center">
                    <div><img :src="'icons/'+day.weather[0].icon+'.png'" alt="icon" width="32" height="32"></div>
                    <div class="ml-3">{{day.weather[0].description}}</div>
                </div>
                <div class="w-1/6 text-right">
                    <div>{{Math.round(day.main.temp_min)-1}}°C</div>
                    <div>{{Math.round(day.main.temp_max)+1}}°C</div>
                </div>
            </div>
        </div> <!-- end future-weather -->

        </div> <!-- end weather-container -->
    </div>
</template>

<script>
    export default {
        mounted() {
            this.fetchData()

            var placesAutocomplete = places({
                appId: 'plPS9O3LEBLU',
                apiKey: '1fa190c53ff8c6b4219e84892bbed141',
                container: document.querySelector('#city'),
                templates: {
                value: function(suggestion) {
                    return suggestion.name;
                    }
                }
            }).configure({
                type: 'city',
                aroundLatLngViaIP: false,
            });

            var $address = document.querySelector('#address-value')
                placesAutocomplete.on('change', (e) => {
                $address.textContent = e.suggestion.value

                this.location.name = `${e.suggestion.name}, ${e.suggestion.country}`
            });

            placesAutocomplete.on('clear', function() {
                $address.textContent = 'none';
            });
        },
        watch: {
            location: {
                handler(newValue, oldValue) {
                    this.fetchData()
                },
                deep: true
            }
        },
        data() {
            return {
                currentTemperature: {
                    actual: '',
                    feels: '',
                    summary: '',
                    icon: '',
                },
                daily: [],
                location: {
                    name: 'Vilnius, Lithuania',
                }
            }
        },
        methods: {
            fetchData() {
                fetch(`/api/weather?city=${this.location.name}`)
                .then(response => response.json())
                .then(data => {
                    this.currentTemperature.actual = Math.round(data.list[0].main.temp)
                    this.currentTemperature.feels = Math.round(data.list[0].main.feels_like)
                    this.currentTemperature.summary = data.list[0].weather[0].description
                    this.currentTemperature.icon = data.list[0].weather[0].icon

                    this.daily = data.list
                })
            },
            toDayOfWeek(timeStamp) {
                const newDate = new Date(timeStamp*1000)
                const days = ['VII.', 'I.', 'II.', 'III.', 'IV.', 'V.', 'VI.',]

                return days[newDate.getDay()]
            }
        }
    }
</script>
