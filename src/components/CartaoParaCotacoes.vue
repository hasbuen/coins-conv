<template>
  <v-container class="pa-4" fluid>
    <!-- Seletor de tema (persistido) -->
    <div class="d-flex justify-end mb-3">
      <v-select
        v-model="tema"
        :items="temaOptions"
        label="Tema"
        density="compact"
        variant="solo-filled"
        hide-details
        style="max-width: 160px"
      />
    </div>

    <!-- Grid responsivo: 3 por linha desktop, 2 tablet, 1 mobile -->
    <v-row align="stretch" justify="center" class="d-flex flex-wrap" no-gutters>
      <v-col
        v-for="moeda in moedas"
        :key="moeda"
        cols="12"
        sm="6"
        md="4"
        class="d-flex justify-center pa-2"
      >
        <v-card
          class="text-center rounded-xl elevation-3 pa-4 w-100"
          :class="isDark ? 'bg-primary text-white' : 'bg-grey-lighten-5 text-grey-darken-4'"
        >
          <v-card-item class="text-h6 font-weight-medium pb-2">
            {{ moeda }}
          </v-card-item>

          <v-card-text class="d-flex align-center justify-center flex-column flex-sm-row gap-2 py-4">
            <v-icon :icon="icone(moeda)" size="42" color="secondary" />
            <div class="text-h4 font-weight-bold">
              {{ cotacoes[moeda] ?? '...' }}
            </div>
          </v-card-text>

          <v-divider class="mx-4 opacity-50"></v-divider>

          <v-card-text>
            <v-text-field
              :model-value="valores[moeda]"
              @update:model-value="val => onInput(val, moeda)"
              :loading="loading[moeda]"
              density="comfortable"
              variant="solo-filled"
              :label="`Valor em ${nomeMoeda(moeda)}`"
              :prefix="prefixSimbolo(moeda)"
              append-inner-icon="mdi-calculator-variant-outline"
              single-line
              hide-details
              color="secondary"
              class="mb-2"
              @click:append-inner="() => converter(moeda)"
            />

            <v-slide-y-transition>
              <div v-if="convertidos[moeda]" class="text-body-1 mt-2">
                {{ moeda }} {{ prefixSimbolo(moeda) }}{{ valores[moeda] || 0 }} × BRL R${{ cotacoes[moeda] }} =
                <strong>R${{ convertidos[moeda] }}</strong>
              </div>
            </v-slide-y-transition>
          </v-card-text>

          <v-progress-linear
            v-if="loading[moeda]"
            indeterminate
            color="secondary"
            height="3"
          />
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
import { useTheme } from "vuetify";

export default {
  name: "CurrencyCards",
  setup() {
    const theme = useTheme();
    return { theme };
  },
  data() {
    return {
      temaOptions: ["Sistema", "Claro", "Escuro"],
      tema: "Sistema",
      moedas: ["USD", "BTC", "EUR"],

      // objetos reativos por moeda
      cotacoes: {},       // ex: { USD: "5.53", BTC: "650699.00" }
      valores: {},        // ex: { USD: "2", BTC: "1" }
      convertidos: {},    // ex: { USD: "11.06" }
      loading: {},        // ex: { USD: false, BTC: false }
    };
  },
  computed: {
    isDark() {
      // leitura segura do tema
      return this.theme.global.current.value && this.theme.global.current.value.dark;
    },
  },
  watch: {
    tema(novo) {
      this.atualizarTema(novo);
    },
  },
  mounted() {
    this.carregarTema();
    this.inicializarEstados();
    this.buscarTodasCotacoes();
  },
  methods: {
    inicializarEstados() {
      // inicializa objetos para cada moeda (evita undefined)
      this.moedas.forEach((m) => {
        this.$set(this.cotacoes, m, null);
        this.$set(this.valores, m, "");
        this.$set(this.convertidos, m, "");
        this.$set(this.loading, m, false);
      });
    },

    async buscarTodasCotacoes() {
      // busca em paralelo, trata erros individualmente
      await Promise.all(this.moedas.map(async (m) => {
        try {
          const resp = await axios.get(`https://economia.awesomeapi.com.br/json/last/${m}-BRL`);
          const key = `${m}BRL`;
          const high = resp?.data?.[key]?.high;
          if (high) {
            // formata com 2 casas ao exibir
            this.$set(this.cotacoes, m, parseFloat(high).toFixed(2));
          } else {
            this.$set(this.cotacoes, m, "0.00");
          }
        } catch (e) {
          console.error(`Erro ao buscar ${m}:`, e);
          this.$set(this.cotacoes, m, "0.00");
        }
      }));
    },

    // chamada quando usuário digita (v-text-field usando model-value)
    onInput(valor, moeda) {
      // aceita somente números / ponto / vírgula
      const cleaned = String(valor).replace(",", ".").replace(/[^\d.]/g, "");
      this.$set(this.valores, moeda, cleaned);
      // converte automaticamente
      if (cleaned && this.cotacoes[moeda]) {
        this.converter(moeda);
      } else {
        this.$set(this.convertidos, moeda, "");
      }
    },

    converter(moeda) {
      const v = parseFloat(this.valores[moeda]);
      const c = parseFloat(this.cotacoes[moeda]);
      if (Number.isNaN(v) || Number.isNaN(c)) return;

      // loading por moeda
      this.$set(this.loading, moeda, true);

      // pequena debounce visual
      setTimeout(() => {
        const result = v * c;
        this.$set(this.convertidos, moeda, result.toFixed(2));
        this.$set(this.loading, moeda, false);
      }, 300);
    },

    icone(moeda) {
      return { USD: "mdi-currency-usd", EUR: "mdi-currency-eur", BTC: "mdi-bitcoin" }[moeda] || "mdi-cash";
    },

    nomeMoeda(moeda) {
      return { USD: "Dólar", EUR: "Euro", BTC: "Bitcoin" }[moeda] || moeda;
    },

    prefixSimbolo(moeda) {
      return { USD: "$", EUR: "€", BTC: "₿" }[moeda] || "";
    },

    // tema (simples e persistente)
    atualizarTema(opcao) {
      const prefersDark =
        window.matchMedia && window.matchMedia("(prefers-color-scheme: dark)").matches;

      if (opcao === "Escuro") this.theme.global.name.value = "dark";
      else if (opcao === "Claro") this.theme.global.name.value = "light";
      else this.theme.global.name.value = prefersDark ? "dark" : "light";

      localStorage.setItem("temaEscolhido", opcao);
    },

    carregarTema() {
      const salvo = localStorage.getItem("temaEscolhido");
      if (salvo && this.temaOptions.includes(salvo)) {
        this.tema = salvo;
      } else {
        this.tema = "Sistema";
      }
      this.atualizarTema(this.tema);
    },
  },
};
</script>

<style scoped>
.v-card {
  transition: transform 0.18s ease, box-shadow 0.18s ease;
}
.v-card:hover {
  transform: translateY(-4px);
}
.v-row {
  row-gap: 20px;
}
/* mobile: torna card mais compacto */
@media (max-width: 600px) {
  .text-h4 {
    font-size: 1.5rem !important;
  }
}
</style>
