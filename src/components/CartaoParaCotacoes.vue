<template>
<v-card
    class="mx-auto bg-primary"
    max-width="278"
  >
    <v-card-item>{{ currency }}</v-card-item>

    <v-card-text class="py-6">
      <v-row align="center" no-gutters>
      <v-icon
          icon="mdi-currency-usd"
          size="40"
          color="secondary"
          class="me-1 pb-1"
        ></v-icon>
        <v-col
          class="text-h4"
          cols="8"
        >
        {{ cotacao }}
        </v-col>
      </v-row>
    </v-card-text>

    <div class="d-flex py-0 justify-space-between">
      <v-list-item>
        <v-list-item-subtitle>
        {{ currency }}-${{ this.valor }} x BRL-R${{ this.cotacao }} = R${{ this.convertido }}
        </v-list-item-subtitle>
      </v-list-item>
    </div>

     <v-card-text>
      <v-text-field
        v-model='valor'
        :loading="loading"
        density="compact"
        variant="solo"
        label="Conversão..."
        prefix="$"
        append-inner-icon="mdi-calculator-variant-outline"
        single-line
        hide-details
        @click:append-inner="onClick"
      ></v-text-field
>
    </v-card-text>
  </v-card>
</template>

<script>
import axios from 'axios'

  export default {
   props: {
    currency: {
      type: String,
    }
  },
    data: () => ({
      loaded: false,
      loading: false,
      cotacao: '',
      valor: '',
      convertido: ''
    }),
  mounted () {
    if(this.currency == 'USD'){
        axios.get(`http://economia.awesomeapi.com.br/json/last/${this.currency}-BRL`).then(response => {
            this.cotacao = response.data.USDBRL.high;
          }).catch(error => {
            console.error(error)
        })
    }

    if(this.currency == 'BTC'){
        axios.get(`http://economia.awesomeapi.com.br/json/last/${this.currency}-BRL`).then(response => {
            this.cotacao = response.data.BTCBRL.high;
          }).catch(error => {
            console.error(error)
        })
    }

      if(this.currency == 'EUR'){
        axios.get(`http://economia.awesomeapi.com.br/json/last/${this.currency}-BRL`).then(response => {
            this.cotacao = response.data.EURBRL.high;
          }).catch(error => {
            console.error(error)
        })
    }

    },
    methods: {
      onClick () {
        this.loading = true
        this.convertido = this.valor * this.cotacao
        this.convertido = this.convertido.toFixed(4)

        setTimeout(() => {
          this.loading = false
          this.loaded = true
        }, 2000)
      },
    },
  }
</script>