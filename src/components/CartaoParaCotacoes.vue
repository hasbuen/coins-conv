<template>
  <v-container class="pa-2" fluid>
    <v-card
      class="mx-auto bg-primary text-center rounded-xl elevation-3"
      max-width="420"
    >
      <v-card-item class="text-h6 text-white font-weight-medium py-3">
        {{ currency }}
      </v-card-item>

      <v-card-text class="d-flex justify-center align-center py-4">
        <v-icon
          :icon="icon"
          size="42"
          color="secondary"
          class="me-2"
        />
        <div class="text-h4 font-weight-bold text-white">
          {{ cotacao }}
        </div>
      </v-card-text>

      <v-divider class="mx-4 opacity-50"></v-divider>

      <v-card-text>
        <v-text-field
          v-model="valor"
          :loading="loading"
          density="comfortable"
          variant="solo-filled"
          label="Valor em {{ currency }}"
          prefix="$"
          append-inner-icon="mdi-calculator-variant-outline"
          single-line
          hide-details
          color="secondary"
          class="mb-2"
          @click:append-inner="converter"
        />

        <v-slide-y-transition>
          <div v-if="convertido" class="text-body-1 mt-2 text-white">
            {{ currency }} ${{ valor || 0 }} × BRL R${{ cotacao }} =
            <strong>R${{ convertido }}</strong>
          </div>
        </v-slide-y-transition>
      </v-card-text>

      <v-progress-linear
        v-if="loading"
        indeterminate
        color="secondary"
        height="3"
      ></v-progress-linear>
    </v-card>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  props: {
    currency: {
      type: String,
      required: true,
    },
  },
  data: () => ({
    loaded: false,
    loading: false,
    cotacao: "",
    valor: "",
    convertido: "",
  }),
  computed: {
    icon() {
      const map = {
        USD: "mdi-currency-usd",
        EUR: "mdi-currency-eur",
        BTC: "mdi-bitcoin",
      };
      return map[this.currency] || "mdi-cash";
    },
  },
  mounted() {
    this.buscarCotacao();
  },
  watch: {
    valor() {
      if (this.valor && this.cotacao) this.converter();
    },
  },
  methods: {
    async buscarCotacao() {
      try {
        const response = await axios.get(
          `https://economia.awesomeapi.com.br/json/last/${this.currency}-BRL`
        );
        const key = `${this.currency}BRL`;
        this.cotacao = parseFloat(response.data[key].high).toFixed(2);
      } catch (error) {
        console.error("Erro ao buscar cotação:", error);
      }
    },
    converter() {
      if (!this.valor || !this.cotacao) return;
      this.loading = true;

      setTimeout(() => {
        const result = parseFloat(this.valor) * parseFloat(this.cotacao);
        this.convertido = result.toFixed(2);
        this.loading = false;
        this.loaded = true;
      }, 500);
    },
  },
};
</script>

<style scoped>
.v-card {
  transition: all 0.3s ease;
}
.v-card:hover {
  transform: scale(1.02);
}
@media (max-width: 600px) {
  .v-card {
    max-width: 95% !important;
  }
  .text-h4 {
    font-size: 1.8rem !important;
  }
}
</style>
