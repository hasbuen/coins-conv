<template>
  <v-container class="pa-4" fluid>
    <v-row align="stretch" justify="center" class="d-flex flex-wrap" no-gutters>
      <v-col cols="12" sm="6" md="4" class="d-flex justify-center pa-2">
        <v-card
          class="text-center rounded-xl elevation-3 pa-4 w-100"
          :class="isDark ? 'bg-primary text-white' : 'bg-grey-lighten-5 text-grey-darken-4'"
        >
          <v-card-item class="text-h6 font-weight-medium pb-2">
            {{ currency }}
          </v-card-item>

          <v-card-text class="d-flex align-center justify-center flex-column flex-sm-row gap-2 py-4">
            <v-icon :icon="icone(currency)" size="42" color="secondary" />
            <div class="text-h4 font-weight-bold">
              {{ cotacao ?? '...' }}
            </div>
          </v-card-text>

          <v-divider class="mx-4 opacity-50"></v-divider>

          <v-card-text>
            <v-text-field
              v-model="valor"
              density="comfortable"
              variant="solo-filled"
              :label="`Valor em ${nomeMoeda(currency)}`"
              :prefix="prefixSimbolo(currency)"
              append-inner-icon="mdi-calculator-variant-outline"
              single-line
              hide-details
              color="secondary"
              class="mb-2"
              @click:append-inner="converter"
            />
            <v-slide-y-transition>
              <div v-if="convertido" class="text-body-1 mt-2">
                {{ currency }} {{ prefixSimbolo(currency) }}{{ valor || 0 }} × BRL R${{ cotacao }} =
                <strong>R${{ convertido }}</strong>
              </div>
            </v-slide-y-transition>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
import { useTheme } from "vuetify";

export default {
  name: "CartaoParaCotacoes",
  props: {
    currency: {
      type: String,
      required: true,
    },
  },
  setup() {
    const theme = useTheme();
    return { theme };
  },
  data() {
    return {
      cotacao: null,
      valor: "",
      convertido: "",
    };
  },
  computed: {
    isDark() {
      return this.theme.global.current.value.dark;
    },
  },
  mounted() {
    this.buscarCotacao();
  },
  methods: {
    async buscarCotacao() {
      try {
        const resp = await axios.get(`https://economia.awesomeapi.com.br/json/last/${this.currency}-BRL`);
        const key = `${this.currency}BRL`;
        this.cotacao = parseFloat(resp.data[key].high).toFixed(2);
      } catch (e) {
        this.cotacao = "0.00";
      }
    },
    converter() {
      const v = parseFloat(this.valor);
      const c = parseFloat(this.cotacao);
      if (!isNaN(v) && !isNaN(c)) {
        this.convertido = (v * c).toFixed(2);
      }
    },
    icone(m) {
      return { USD: "mdi-currency-usd", EUR: "mdi-currency-eur", BTC: "mdi-bitcoin" }[m] || "mdi-cash";
    },
    nomeMoeda(m) {
      return { USD: "Dólar", EUR: "Euro", BTC: "Bitcoin" }[m] || m;
    },
    prefixSimbolo(m) {
      return { USD: "$", EUR: "€", BTC: "₿" }[m] || "";
    },
  },
};
</script>
