<template>
  <div class="container">
    <div class="d-flex mt-4">
      <div class="filter">
        <div class="options">
          <div class="title" style="display: flex; justify-content: space-between">
            <div>Опции тарифа</div>
            <a 
              href="#" 
              type="button" 
              v-show="disabledTypes" 
              @click.prevent="clearTypes" 
              title="Сбросить выбор">
                <img src="icon-close-new.svg" alt="Сбросить выбор">
            </a>
          </div>
          <div class="option-list">
            <div class="list-item" v-for="(type, index) of types" :key="type.value">
              <label class="d-flex checkbox-block">
                <input
                  type="checkbox"
                  :id="'checkbox' + index"
                  :value="type.value"
                  :disabled="disabledTypes"
                  v-model="filterType"
                />
                <span></span>
                {{ type.name }}
              </label>
            </div>
          </div>
        </div>
        <div class="aircompanies">
          <div class="title">Авиакомпании</div>
          <div class="aircompany-list">
            <div class="list-item">
              <label class="d-flex checkbox-block">
                <input
                  type="checkbox"
                  id="checkbox"
                  v-model="showAll"
                  @change="selectAll"
                />
                <span></span>
                Все
              </label>
            </div>
            <div
              class="list-item"
              v-for="(airline, index) of airlines"
              :key="airline.key"
            >
              <label class="d-flex checkbox-block">
                <input
                  type="checkbox"
                  :id="'checkbox' + index"
                  :value="airline.key"
                  v-model="filterAirlines"
                />
                <span></span>
                {{ airline.name}}
              </label>
            </div>
          </div>
        </div>
        <div class="drop-filter">
          <a @click.prevent="clearAllFilters">Сбросить все фильтры</a>
        </div>
      </div>
      <div class="tickets">
        <div
          class="ticket flex-1"
          v-for="flight in filteredFlights"
          :key="flight.id"
        >
          <div class="details-block">
            <div class="flex-center">
              <div class="logo flex-center-2">
                <img
                  :src="
                    'https://aviata.kz/static/airline-logos/80x80/' +
                    flight.carrier +
                    '.png'
                  "
                  alt="aircompany"
                />
                <div class="logo-text">
                  {{ flight.carrierName }}
                </div>
              </div>
              <div class="flex-center-2">
                <div class="start-date">
                  <div class="date">{{ flight.depDate }} ноя, вс</div>
                  <div class="time">{{ flight.depTime }}</div>
                </div>
                <div class="distance-detail">
                  <div class="detail-text">через Шымкент, 1ч 50м</div>
                </div>
                <div class="end-date">
                  <div class="date">{{ flight.arrDate }} ноя, вс</div>
                  <div class="time">{{ flight.arrTime }}</div>
                </div>
              </div>
            </div>
            <div class="actions d-flex align-items-center">
              <div class="more-info">
                <a href="#">Детали перелета</a>
                <a href="#">Условия тарифа</a>
              </div>
              <div class="no-return">
                <a href="#">{{ flight.refundable ? 'возвратный' : 'невозвратный' }}</a>
              </div>
            </div>
          </div>
          <div class="price-block text-center">
            <div class="ticket-price">{{ flight.price }} т</div>
            <div class="select-ticket text-center">
              <button>Выбрать</button>
            </div>
            <div class="price-text">Цена за всех пассажиров</div>
            <div class="flex-center-2">
              <div class="no-baggage">{{ flight.baggage > 0 ? "Есть багаж" : "Нет багажа" }}</div>
              <div class="add-baggage">
                <button>+ Добавить багаж</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
    name: 'AirTickets',
    props: {
        air_tickets: Object
    },
    data() {
        return {
            filterAirlines: [],
            showAll: true,
            filterType: [],
            types: [
                {
                    name: 'Только прямые', 
                    value: 1
                },
                {
                    name: 'Только с багажом',
                    value: 2 
                },
                {
                    name: 'Только возвратные',
                    value: 3
                }
            ],
        }
    },
    methods: {
        clearAllFilters() {
          this.showAll = true;
          this.filterType = [];
          this.filterAirlines = [];
        }, 
        clearTypes() {
          this.filterType = []
        }
    },
    computed: {
        flightsData() {
            const flights = [];
            this.air_tickets.flights.forEach((flight) => {
                const flightData = {
                  id: flight.id,
                  price: flight.price,
                  carrierName: flight.itineraries[0][0].carrier_name,
                  carrier: flight.itineraries[0][0].carrier,
                  depDate: flight.itineraries[0][0].dep_date.substr(7, 2),
                  depTime: flight.itineraries[0][0].dep_date.substr(10, 5),
                  arrDate: flight.itineraries[0][0].arr_date.substr(7, 2),
                  arrTime: flight.itineraries[0][0].arr_date.substr(10, 5),
                  refundable: flight.itineraries[0][0].refundable,
                  stops: flight.itineraries[0][0].stops,
                  baggage: flight.itineraries[0][0].segments[0].baggage_options[0].value
                }
                flights.push(flightData)
            })
            return flights 
        },
        airlines() {
            const modifiedAirlines = []
            for (var key in this.air_tickets.airlines) {
                const airline = {
                    key: key,
                    name: this.air_tickets.airlines[key]
                }
                modifiedAirlines.push(airline)
            }
            return modifiedAirlines;
        },
        filteredFlights() {
           let filtered = this.flightsData.filter(fl => this.filterAirlines.includes(fl.carrier)) 
           filtered = this.filterAirlines.length ? filtered : (filtered = this.flightsData)
           if (this.filterType == 1) {
              return filtered.filter(fl => fl.stops === 0)
           }
           if (this.filterType == 2) {
              return filtered.filter(fl => fl.baggage !== 0)
           }
           if (this.filterType == 3) {
               return filtered.filter(fl => fl.refundable === true)
           }
           if (!this.filterAirlines.length) { 
               this.showAll = true
               return this.flightsData 
            }
           if (this.filterAirlines.length) {
               this.showAll = false
               return filtered 
           } 
        },
        disabledTypes() {
          if (this.filterType.length > 0) return true
        }
    },
}
</script>

<style>
</style>