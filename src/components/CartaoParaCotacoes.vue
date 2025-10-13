<template>
  <v-card
    class="pa-4 d-flex flex-column align-center justify-center"
    :color="cardColor"
    elevation="6"
    rounded="xl"
  >
    <!-- Título da moeda -->
    <div class="text-h6 font-weight-bold mb-2 text-center">
      {{ currencyName }}
    </div>

    <!-- Valor principal -->
    <div class="d-flex align-center mb-3">
      <v-icon :icon="currencyIcon" size="32" class="me-2" color="teal-accent-3" />
      <span class="text-h5 font-weight-bold">{{ valorFormatado }}</span>
    </div>

    <!-- Texto auxiliar -->
    <div class="text-caption text-center text-medium-emphasis mb-1">
      Valor em Real (BRL)
    </div>

    <!-- Botão para simular conversão -->
    <v-btn
      icon
      variant="text"
      color="white"
      @click="converterExemplo"
    >
      <v-icon>mdi-calculator-variant</v-icon>
    </v-btn>

    <!-- Resultado da conversão -->
    <div v-if="resultado" class="text-body-2 mt-3 text-center font-mono">
      {{ resultado }}
    </div>
  </v-card>
</template>

<script>
export default {
  name: "CartaoParaCotacoes",
  props: {
    currency: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      valor: 0,
      resultado: "",
    };
  },
  computed: {
    cardColor() {
      return this.$vuetify.theme.global.current.value.dark
        ? "deep-purple-accent-2"
        : "deep-purple-lighten-3";
    },
    currencyName() {
      const nomes = {
        USD: "Dólar Americano",
        EUR: "Euro",
        BTC: "Bitcoin",
      };
      return nomes[this.currency] || this.currency;
    },
    currencyIcon() {
      const icons = {
        USD: "mdi-currency-usd",
        EUR: "mdi-currency-eur",
        BTC: "mdi-bitcoin",
      };
      return icons[this.currency] || "mdi-cash";
    },
    valorFormatado() {
      const valoresExemplo = {
        USD: 5.53,
        EUR: 6.41,
        BTC: 650699.0,
      };
      const valor = valoresExemplo[this.currency] || 0;
      return valor.toLocaleString("pt-BR", {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      });
    },
  },
  methods: {
    converterExemplo() {
      const valorBRL = 100;
      const taxa = {
        USD: 5.53,
        EUR: 6.41,
        BTC: 650699.0,
      }[this.currency];
      const total = valorBRL * taxa;
      this.resultado = `R$ ${valorBRL} × ${this.currency} ${taxa} = R$${total.toLocaleString(
        "pt-BR",
        { maximumFractionDigits: 2 }
      )}`;
    },
  },
};
</script>

<style scoped>
.v-card {
  width: 100%;
  max-width: 300px;
  transition: all 0.3s ease;
}
.v-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 18px rgba(0, 0, 0, 0.2);
}
</style>
