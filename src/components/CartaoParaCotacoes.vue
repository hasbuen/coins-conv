<template>
  <v-container class="pa-4" fluid>
    <!-- Seletor de tema -->
    <div class="d-flex justify-end mb-3">
      <v-select
        v-model="tema"
        :items="['Sistema', 'Claro', 'Escuro']"
        label="Tema"
        density="compact"
        variant="solo-filled"
        hide-details
        class="theme-select"
        style="max-width: 160px"
      />
    </div>

    <!-- Card principal -->
    <v-card
      class="mx-auto text-center rounded-xl elevation-3 pa-4 transition-all"
      max-width="420"
      :class="isDark ? 'bg-primary text-white' : 'bg-grey-lighten-5 text-grey-darken-4'"
    >
      <v-card-item class="text-h6 font-weight-medium pb-2">
        {{ currency }}
      </v-card-item>

      <v-card-text
        class="d-flex align-center justify-center flex-column flex-sm-row gap-2 py-4"
      >
        <v-icon :icon="icon" size="42" color="secondary" />
        <div class="text-h4 font-weight-bold">
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
          :label="`Valor em ${moedaNome}`"
          prefix="$"
          append-inner-icon="mdi-calculator-variant-outline"
          single-line
          hide-details
          color="secondary"
          class="mb-2"
          @click:append-inner="converter"
        />

        <v-slide-y-transition>
          <div v-if="convertido" class="text-body-1 mt-2">
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
import { useTheme } from "vuetify";

export default {
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
  data: () => ({
    tema: "Sistema",
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
    moedaNome() {
      const nomes = {
        USD: "Dólar",
        EUR: "Euro",
        BTC: "Bitcoin",
      };
      return nomes[this.currency] || this.currency;
    },
    isDark() {
      return this.theme.global.current.value.dark;
    },
  },
  watch: {
    tema(novo) {
      this.atualizarTema(novo);
    },
    valor() {
      if (this.valor && this.cotacao) this.converter();
    },
  },
  mounted() {
    this.buscarCotacao();
    this.carregarTema();
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
      }, 400);
    },
    atualizarTema(opcao) {
      const prefersDark =
        window.matchMedia &&
        window.matchMedia("(prefers-color-scheme: dark)").matches;

      if (opcao === "Escuro") this.theme.global.name.value = "dark";
      else if (opcao === "Claro") this.theme.global.name.value = "light";
      else this.theme.global.name.value = prefersDark ? "dark" : "light";

      localStorage.setItem("temaEscolhido", opcao);
    },
    carregarTema() {
      const salvo = localStorage.getItem("temaEscolhido");
      if (salvo) {
        this.tema = salvo;
        this.atualizarTema(salvo);
      } else {
        this.atualizarTema("Sistema");
      }
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
.theme-select {
  transition: all 0.3s ease;
}
@media (max-width: 600px) {
  .v-card {
    max-width: 95% !important;
    text-align: left !important;
  }
  .v-card-text.flex-sm-row {
    flex-direction: row !important;
    justify-content: space-between !important;
  }
  .text-h4 {
    font-size: 1.8rem !important;
  }
}
</style>
