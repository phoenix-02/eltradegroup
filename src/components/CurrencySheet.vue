<template>
  <div>
    <link href="//fonts.googleapis.com/css?family=Open+Sans:300,400,600,700&amp;subset=latin" rel="stylesheet">
    <v-card
      id="currency_widget"
      color="amber lighten-3 "
    >
      <v-card-title class=" py-6">
        <h4 class="font-weight-bold">
          Курс на {{ date }}
        </h4>
      </v-card-title>
      <v-tabs
        v-model="tab"
        background-color="amber lighten-3"
        color="disabled"
        center-active
        icons-and-text
        show-arrows
      >


        <v-tabs-slider/>
        <v-tab
          v-for="name in all_currency_names" :key="name"
          v-on:click="getCurrency(name)"
          :href="'#tab-'+name">{{ name }}

        </v-tab>
      </v-tabs>
      <v-tabs-items v-model="tab">

        <v-tab-item :key="base_currency.toString()"
                    :value="'tab-'+base_currency.toString()">
          <v-row class="px-4" justify="space-between">
            <v-col
              cols="6"
              sm="4"
              md="3"
            >
              <v-menu
                v-model="date_menu"
              >
                <template v-slot:activator="{ on }">
                  <v-text-field
                    readonly
                    prefix="📆"
                    :value="date"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  locale="ru-ru"
                  v-model="date"
                  no-title
                  @change="getCurrency(base_currency)"
                ></v-date-picker>
              </v-menu>
            </v-col>
            <v-col
              cols="6"
              sm="6"
              md="3"
            >
              <v-text-field v-model="base_currency_amount" :suffix="base_currency" label="Введите сумму" type="number">
                5
              </v-text-field>

            </v-col>
          </v-row>
          <v-row justify="center">
            <v-col
              cols="10"
              sm="5"
              md="5"
              lg="5"
              v-for="currency in changePaginatedData" :key="currency[0].toString()" :value="'tab-'+currency[0]">
              <v-card color="dark">
                <v-card-title>
                  {{ base_currency_amount }}<span class="text--disabled"> {{ base_currency }}=</span>
                </v-card-title>

                <v-card-text class="currency_convert">
                  <span class="currency_convert_result">{{ (currency[1] * base_currency_amount).toFixed(2) }}</span>
                  <span class="currency_convert_symbol">{{ currency[0] }}</span>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>

        </v-tab-item>
        <div class="controls ma-4">
          <v-btn v-on:click="page--" :disabled="page<=1">˂ назад</v-btn>
          <v-btn v-on:click="page++" :disabled="Object.keys(all_currency).length / currency_per_page <= page ">далее
            &#707;
          </v-btn>
        </div>
      </v-tabs-items>

    </v-card>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'CurrencySheet',
  data: () => ({
    page: 1,
    tab: null,
    date: 'latest',
    date_menu: false,
    all_currency: null,
    all_currency_names: [],
    currency_per_page: 4,
    paginated_currency: [],
    base_currency: String,
    base_currency_amount: 5,
    api_key: 'c52a392eb94633dd0581b8cec4a15655',
  }),
  methods: {
    getCurrency(currency = 'EUR') {
      axios
        .get(`https://api.openrates.io/${this.date}?acceess_key=${this.api_key}&base=${currency}`)
        .then(response => {
          this.date = response.data['date']
          this.base_currency = response.data['base']
          this.all_currency = response.data['rates']
          // необходимо для добавления евро во вкладки, т к сервис отдает именно эту валюту отдельно от остальных,
          // если она стоит базовой
          this.all_currency_names = Object.keys(this.all_currency)
          if (!this.all_currency_names.includes(this.base_currency)) {
            this.all_currency_names.unshift(this.base_currency)
          }
          this.all_currency_names.sort()

          // скрываю курс валюты к самой себе
          delete this.all_currency[this.base_currency];
          // default tab
          this.tab = 'tab-' + this.base_currency.toString()

        })
    },
  },
  computed:{
    changePaginatedData(){
      return Object.entries(this.all_currency).slice(this.page * this.currency_per_page - this.currency_per_page, this.page * this.currency_per_page)
    }
  },
  beforeMount() {
    this.getCurrency()
  },
}
</script>

<style scoped>
.v-tab--active {
  background-color: #fff;
  border-radius: 10% 10% 0 0;
}

#currency_widget {
  max-width: 720px;
  min-width: 320px;
}

.currency_convert {
  font-family: 'open sans', arial, sans-serif;
  color: #2B2D33 !important;
  font-weight: 300 !important;
}

.currency_convert_result {
  font-size: 2.5rem;
}

.currency_convert_symbol {
  font-size: 1.5rem
}

.controls button {
  margin: 15px;
}

</style>
