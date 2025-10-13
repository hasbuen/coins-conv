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
        style="max-width: 160px"
      />
    </div>

    <!-- Linha responsiva de cards -->
    <v-row
      align="stretch"
      justify="center"
      class="d-flex flex-wrap"
      no-gutters
    >
      <v-col
        v-for="moeda in moedas"
        :key="moeda"
        cols="12"
        sm="6"
        md="4"
        class="d-flex justify-center pa-2"
      >
        <!-- Card de cotação -->
        <v-card
          class="text-center rounded-xl elevation-3 pa-4 transition-all w-100"
          :class="isDark ? 'bg-primary text-white' : 'bg-grey-lighten-5 text-grey-darken-4'"
        >
          <v-card-item class="text-h6 font-weight-medium pb-2">
            {{ moeda }}
          </v-card-item>

          <v-card-text
            class="d-flex align-center justify-center flex-column flex-sm-row gap-2 py-4"
          >
            <v-icon :icon="icone(moeda)" size="42" color="secondary" />
            <div class="text-h4 font-weight-bold">
              {{ cotacoes[moeda] || '...' }}
            </div>
          </v-card-text>

          <v-divider class="mx-4 opacity-50"></v-divider>

          <v-card-text>
            <v-text-field
              v-model="valores[moeda]"
              :loading="loading"
              density="comfortable"
              variant="solo-filled"
              :label="`Valor em ${nomeMoeda(moeda)}`"
              prefix="$"
              append-inner-icon="mdi-calculator-variant-outline"
              single-line
              hide-details
              color="secondary"
              class="mb-2"
              @click:append-inner="() => converter(moeda)"
            />

            <v-slide-y-transition>
              <div
                v-if="convertidos[moeda]"
                class="text-body-1 mt-2"
              >
                {{ moeda }} ${{ valores[moeda] || 0 }} × BRL R${{ cotacoes[moeda] }} =
                <strong>R${{ convertidos[moeda] }}</strong>
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
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
import { useTheme } from "vuetify";

export default {
  setup() {
    const theme = useTheme();
    return { theme };
  },
  data: () => ({
    tema: "Sistema",
    moedas: ["USD", "BTC", "EUR"],
    cotacoes: {},
    valores: {},
    convertidos: {},
    loading: false,
  }),
  computed: {
    isDark() {
      return this.theme.global.current.value.dark;
    },
  },
  watch: {
    tema(novo) {
      this.atualizarTema(novo);
    },
  },
  mounted() {
    this.carregarTema();
    this.buscarTodasCotacoes();
  },
  methods: {
    async buscarTodasCotacoes() {
      for (const moeda of this.moedas) {
        try {
          const response = await axios.get(
            `https://economia.awesomeapi.com.br/json/last/${moeda}-BRL`
          );
          const key = `${moeda}BRL`;
          this.$set(this.cotacoes, moeda, parseFloat(response.data[key].high).toFixed(2));
        } catch (error) {
          console.error(`Erro ao buscar ${moeda}:`, error);
        }
      }
    },
    converter(moeda) {
      if (!this.valores[moeda] || !this.cotacoes[moeda]) return;
      this.loading = true;
      setTimeout(() => {
        const result =
          parseFloat(this.valores[moeda]) * parseFloat(this.cotacoes[moeda]);
        this.$set(this.convertidos, moeda, result.toFixed(2));
        this.loading = false;
      }, 400);
    },
    icone(moeda) {
      const map = {
        USD: "mdi-currency-usd",
        EUR: "mdi-currency-eur",
        BTC: "mdi-bitcoin",
      };
      return map[moeda] || "mdi-cash";
    },
    nomeMoeda(moeda) {
      const nomes = {
        USD: "Dólar",
        EUR: "Euro",
        BTC: "Bitcoin",
      };
      return nomes[moeda] || moeda;
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

/* 🔹 Responsividade e layout */
.v-row {
  row-gap: 16px;
}
.v-col {
  display: flex;
}
.v-card-text.flex-sm-row {
  flex-direction: row !important;
  justify-content: space-between !important;
}

@media (max-width: 600px) {
  .text-h4 {
    font-size: 1.6rem !important;
  }
}
</style>
