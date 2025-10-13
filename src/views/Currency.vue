<template>
  <v-app>
    <!-- Barra superior -->
    <v-app-bar app flat>
      <v-toolbar-title>Conversor de Moedas</v-toolbar-title>

      <v-spacer></v-spacer>

      <!-- Seletor de tema -->
      <v-select
        v-model="selectedTheme"
        :items="themes"
        variant="outlined"
        density="compact"
        hide-details
        style="max-width: 120px"
        @update:modelValue="changeTheme"
      />
    </v-app-bar>

    <!-- Conteúdo principal -->
    <v-main class="pa-6 d-flex justify-center align-center">
      <v-container>
        <v-row
          align="center"
          justify="center"
          dense
          class="text-center"
        >
          <v-col
            v-for="(item, index) in lista"
            :key="index"
            cols="12"
            sm="6"
            md="4"
            class="d-flex justify-center"
          >
            <CartaoParaCotacoes :currency="item.currency" />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import CartaoParaCotacoes from '@/components/CartaoParaCotacoes.vue'

export default {
  name: 'CurrencyView',
  components: { CartaoParaCotacoes },
  data() {
    return {
      selectedTheme: 'Sistema',
      themes: ['Claro', 'Escuro', 'Sistema'],
      lista: [
        { currency: 'USD' },
        { currency: 'BTC' },
        { currency: 'EUR' },
      ],
    }
  },
  mounted() {
    this.applySystemTheme()
  },
  methods: {
    changeTheme(theme) {
      const isDark = this.$vuetify.theme.global.name.value === 'dark'

      if (theme === 'Claro') {
        this.$vuetify.theme.global.name.value = 'light'
      } else if (theme === 'Escuro') {
        this.$vuetify.theme.global.name.value = 'dark'
      } else {
        this.applySystemTheme()
      }
    },
    applySystemTheme() {
      const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
      this.$vuetify.theme.global.name.value = prefersDark ? 'dark' : 'light'
    },
  },
}
</script>

<style scoped>
.v-application {
  background-color: transparent !important;
}

.v-main {
  min-height: 100vh;
}

.v-app-bar {
  backdrop-filter: blur(12px);
  background-color: rgba(0, 0, 0, 0.3) !important;
}

.v-select {
  color: white !important;
}
</style>
