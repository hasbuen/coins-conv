<template>
  <v-app>
    <!-- Barra de topo com seletor de tema -->
    <v-app-bar flat density="compact" class="px-4">
      <v-toolbar-title class="text-h6">Conversor de Moedas</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-select
        v-model="tema"
        :items="['Sistema', 'Claro', 'Escuro']"
        label="Tema"
        density="compact"
        variant="solo-filled"
        hide-details
        style="max-width: 140px"
      />
    </v-app-bar>

    <v-main>
      <v-container class="py-6" fluid>
        <v-row
          align="stretch"
          justify="center"
          class="d-flex flex-wrap"
          no-gutters
        >
          <!-- 1 card por moeda -->
          <v-col
            v-for="(item, index) in lista"
            :key="index"
            cols="12"
            sm="6"
            md="4"
            class="d-flex justify-center pa-2"
          >
            <CartaoParaCotacoes :currency="item.currency" />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import { useTheme } from "vuetify";
import CartaoParaCotacoes from "@/components/CartaoParaCotacoes.vue";

export default {
  name: "CurrencyView",
  components: {
    CartaoParaCotacoes,
  },
  setup() {
    const theme = useTheme();
    return { theme };
  },
  data() {
    return {
      tema: "Sistema",
      lista: [
        { currency: "USD" },
        { currency: "BTC" },
        { currency: "EUR" },
      ],
    };
  },
  watch: {
    tema(novo) {
      this.aplicarTema(novo);
    },
  },
  mounted() {
    this.carregarTema();
  },
  methods: {
    aplicarTema(opcao) {
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
        this.aplicarTema(salvo);
      } else {
        this.aplicarTema("Sistema");
      }
    },
  },
};
</script>

<style scoped>
.v-main {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}
.v-container {
  max-width: 1200px;
}
</style>
